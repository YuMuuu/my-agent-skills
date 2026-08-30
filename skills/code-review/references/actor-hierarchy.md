# Pekko Typed Actor hierarchy

Actor を利用する場合は、常に Apache Pekko Typed を利用する。Akka Actor または Pekko Classic を新しい実装へ採用しない。

Actor hierarchy とは、Actor の生成関係によって形成されるライフサイクルと障害監督の木構造である。class hierarchy や機能一覧ではない。

## コード生成前

プロジェクトで初めて Actor を実装する場合は、事前確認が必要である。top-level hierarchy をユーザへ確認する。

既存プロジェクトでは、次を確認する。

- Pekko Typed の利用状況
- guardian 配下の hierarchy
- Akka Actor または Pekko Classic の利用有無。使われている場合は規約不一致として報告する

## 設計

- ActorSystem の guardian は、subsystem の boot に集中させる。起動した subsystem のライフサイクルも監視する。
- Actor を spawn した親が、その子のライフサイクルと failure handling に責任を持つ。
- 一緒に起動・停止する Actor を同じ subtree へ配置する。
- 障害を分離したい単位と subtree の境界を一致させる。
- 長く存続する Actor を、短命な Actor の子にしない。
- ActorRef を広範囲へ公開し、親による管理を迂回しない。

```text
ActorSystem / Guardian
├── OrderSubsystem
│   ├── OrderCoordinator
│   └── OrderWorker
├── PaymentSubsystem
│   └── PaymentWorker
└── NotificationSubsystem
    └── NotificationWorker
```

## コード例

次の例では、各 object を同名の Scala ファイルへ定義する。

`Guardian.scala`では、guardian が subsystem を生成して監視する。

```scala
import org.apache.pekko.actor.typed.{Behavior, Terminated}
import org.apache.pekko.actor.typed.scaladsl.Behaviors

object Guardian {
  enum Command {
    case Stop
  }

  def apply(): Behavior[Command] =
    Behaviors.setup { context =>
      val orderSubsystem =
        context.spawn(OrderSubsystem(), "orders")

      context.watch(orderSubsystem)

      Behaviors
        .receiveMessage[Command] {
          case Command.Stop =>
            context.stop(orderSubsystem)
            Behaviors.same
        }
        .receiveSignal {
          case (_, Terminated(ref)) if ref == orderSubsystem =>
            Behaviors.stopped
        }
    }
}
```

`OrderSubsystem.scala`では、subsystem が worker を生成する。worker の failure handling も親が定義する。

```scala
import org.apache.pekko.actor.typed.{Behavior, SupervisorStrategy}
import org.apache.pekko.actor.typed.scaladsl.Behaviors

object OrderSubsystem {
  enum Command {
    case Submit(orderId: OrderId)
  }

  def apply(): Behavior[Command] =
    Behaviors.setup { context =>
      val workerBehavior =
        Behaviors
          .supervise(OrderWorker())
          .onFailure[IllegalStateException](SupervisorStrategy.restart)

      val worker = context.spawn(workerBehavior, "worker")

      Behaviors.receiveMessage {
        case Command.Submit(orderId) =>
          worker ! OrderWorker.Command.Process(orderId)
          Behaviors.same
      }
    }
}
```

`OrderWorker.scala`の worker は、親から委譲された小さな処理を担当する。この例では hierarchy に焦点を当てるため、message 処理を省略する。

```scala
import org.apache.pekko.actor.typed.Behavior
import org.apache.pekko.actor.typed.scaladsl.Behaviors

object OrderWorker {
  enum Command {
    case Process(orderId: OrderId)
  }

  def apply(): Behavior[Command] =
    Behaviors.ignore
}
```

この例の生成関係は`Guardian → OrderSubsystem → OrderWorker`になる。guardian を停止すると、配下の Actor も停止する。worker の障害は、直接の親である`OrderSubsystem`が処理する。

## supervisionと監視

- validation error や not found は、想定可能な失敗である。例外ではなく通常の message protocol で表現する。
- Actor 内部の予期しない failure は、親の supervision 方針で処理する。
- `watch`した Actor の`Terminated`または`ChildFailed`を処理する。
- restart、stop、escalate の選択と、Actor が保持する state の整合性を確認する。
- restart 時に子 Actor、timer、stream、外部 resource を重複生成しないか確認する。
- 親の restart または stop が子 Actor へ与える影響を確認する。

Pekko Typed では、supervision を指定しない Actor が例外を投げると default で停止する。この挙動を前提に failure handling を設計する。

## shutdown

- 親の停止で子が再帰的に停止することを前提に、所有関係を設計する。
- ActorSystem の Coordinated Shutdown と独自の終了処理を競合させない。
- 新規 message の受付停止、処理中 message の完了方針、外部 resource の解放順序を定義する。
- ActorSystem を機能 class の内部へ隠さない。application の boot 層を所有者にする。

## 参考文献

- [Apache Pekko Actor lifecycle](https://pekko.apache.org/docs/pekko/current/typed/actor-lifecycle.html)
- [Apache Pekko Fault Tolerance](https://pekko.apache.org/docs/pekko/current/typed/fault-tolerance.html)
- [Apache Pekko Actor Systems](https://pekko.apache.org/docs/pekko/current/general/actor-systems.html)
