# Scala

Scala コードを生成またはレビューする場合に適用する。build 設定から Scala version を確認し、Scala 3 の規約を Scala 2 へ誤適用しない。

## コード生成前

プロジェクト最初の実装では、次を確認する。既存プロジェクトでは採用済みの方針を調べ、変更する必要がある場合だけユーザーへ確認する。

- software architecture とレイヤー構成
- DI を利用するか。利用する場合はどのライブラリか
- DB アクセスライブラリ
- 非同期境界に`Future`または Cats Effect `IO`のどちらを利用するか
- 設定管理に PureConfig を採用するか。既存方式から変更するか

`java.sql`を直接使う実装を選ばず、ユーザーが選択した DB アクセスライブラリを利用する。

## ファイルと型

- 原則として 1 つの class を 1 つのファイルに定義する。
- class と同名の companion object は、同じファイルへの定義を許可する。
- Scala 3 で閉じた値集合や状態を表現する場合は`enum`を利用する。
- ID、名称、時刻、金額、数量、ステータス、外部識別子には、`opaque type`、case class、value class、enum などを利用する。

```scala
opaque type PersonId = Long

object PersonId {
  def apply(value: Long): PersonId = value

  extension (personId: PersonId)
    def value: Long = personId
}
```

## 分岐

Boolean 値による二分岐には`if`を利用できる。値、状態、型による分岐では`match`を利用し、網羅性を確認する。

```scala
value match {
  case 1 => foo
  case 2 => bar
  case _ => buzz
}
```

## flatMapとfor式

次の場合は for 式を利用する。

- `flatMap`が 2 つ以上続く場合
- 1 つ以上の`flatMap`と`map`を組み合わせる場合

```scala
for {
  user <- findUser(userId)
  order <- findOrder(user)
} yield order
```

## レイヤー境界

ドメインレイヤー境界を越える非同期 interface では、次の形を利用する。採用した effect に応じて型を選ぶ。application 層と infrastructure 層の境界などが該当する。

```scala
Future[Either[Error, Result]]
```

```scala
IO[Either[Error, Result]]
```

Cats は標準ライブラリ相当として扱うため、`IO`は interface へ公開してよい。`EitherT`と`OptionT`は内部合成に使い、interface へ公開しない。

純粋な domain 処理へ不要な`Future`や`IO`を持ち込まない。

```scala
def calculatePrice(
    quantity: Quantity,
    unitPrice: UnitPrice
): Either[PricingError, Price]
```

## import

コード生成後のレビューで、完全修飾名を型の利用箇所へ直接書いていないか確認する。他ファイルの型はファイル先頭で import する。

```scala
import dev.yumuuu.timeline.infrastructure.messaging.PostFanoutEvent

def publish(event: PostFanoutEvent): Unit
```

次の形式は避ける。

```scala
def publish(
    event: dev.yumuuu.timeline.infrastructure.messaging.PostFanoutEvent
): Unit
```

未使用 import、wildcard import、不要な alias、同名型の衝突も確認する。

## 設定とDI

- 設定値の読み込みには PureConfig を利用する。
- ID、期間、URL などのドメイン概念を表す設定値は、検証後にドメイン型へ変換する。
- 不正な設定は、利用箇所まで遅延させず起動時に検出する。
- DI 方式はプロジェクト最初の実装時に確認し、その後は決定した方式へ従う。
- object や global state へ依存を隠さず、boot 層で依存関係を組み立てる。

既存プロジェクトの方式が異なる場合は、最初の実装時にユーザーへ確認する。既存方式の維持または変更を選んでもらう。
