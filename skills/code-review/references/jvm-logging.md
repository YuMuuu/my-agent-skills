# JVM loggingとMDC

SLF4J、Logback、MDC、非同期処理を利用する JVM コードで適用する。

## logger

Scala の class では、logger をインスタンスフィールドとして定義する。

```scala
private val logger = LoggerFactory.getLogger(getClass)
```

## 調査可能なログ

- error log には、障害対象を追跡できる ID を含める。
- trace ID、request ID、entity ID、job ID、message ID など、処理に対応する ID を選ぶ。
- message 文字列だけでなく、検索可能な structured field として記録する。
- password、access token、session ID、個人情報、秘密の SQL parameter を記録しない。

## MDC伝播

MDC は thread local である。`Future`、thread pool、stream、fiber などの非同期境界では、自動的に伝播するとは限らない。

MDC 伝播を実装する前に、対象処理のログへ context が必要か確認する。すべての処理へ application 全体の MDC を伝播する必要はない。request と関連しない background 処理など、MDC が不要な処理には伝播を追加しない。必要性をコードや運用要件から判断できない場合は、ユーザへ確認する。

- 非同期 callback 内でログを出す時点に、正しい context が設定されているか確認する。
- 処理開始前の MDC を保存し、処理後に復元する。
- context が存在しなかった場合は clear し、別 request の値を残さない。
- success、failure、cancellation のすべてで復元処理が実行されるようにする。
- application 全体の thread pool を安易に独自実装で置換せず、framework と effect system に適した伝播方法を選ぶ。
- framework または effect system の組み込み伝播機能を使う場合は、実際の非同期境界で動作を確認する。
- 複数の並行処理の方式、effect system、framework をまたぐ場合は、組み込み機能だけで context が伝播しない可能性を確認する。
- 組み込み機能で伝播経路を保証できない場合は、境界 adapter、executor wrapper、context carrier などの独自実装を利用する。
- 独自実装では、context の設定だけでなく、以前の値の復元と clear を必ず扱う。

request 属性へ trace ID を保持するだけでは、非同期 callback 内の MDC 設定は保証されない。ログを出す実行箇所まで context が到達する経路を確認する。

参考実装: [play-logging-sandboxのMDC修正](https://github.com/YuMuuu/play-logging-sandbox/commit/3175e4bf5a7bf13f227740138cd38211f3b4ef4d)
