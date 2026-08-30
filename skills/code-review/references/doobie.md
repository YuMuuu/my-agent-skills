# Doobie

Doobie を利用する Scala コードで適用する。実装前に、対象プロジェクトで利用している Doobie と Cats Effect の version および既存の mapping 方法を確認する。

## query結果型

`.query`には、tuple やプリミティブ値ではなく、意味を持つ結果型を指定する。SQL と対応する case class の定義も確認する。

```scala
final case class Person(
    id: PersonId,
    name: PersonName
)
```

```scala
val query: ConnectionIO[List[Person]] =
  sql"""
    SELECT id, name
    FROM person
  """.query[Person].to[List]
```

column の順序と型が case class の constructor と一致しているか確認する。

## Meta

domain 型を DB の値へ mapping する場合は、対応する`Meta`を定義する。opaque type、value class、enum などが対象になる。

```scala
opaque type PersonId = Long

object PersonId {
  def apply(value: Long): PersonId = value

  extension (personId: PersonId)
    def value: Long = personId

  given Meta[PersonId] =
    Meta[Long].imap(PersonId.apply)(_.value)
}

opaque type PersonName = String

object PersonName {
  def apply(value: String): PersonName = value

  extension (personName: PersonName)
    def value: String = personName

  given Meta[PersonName] =
    Meta[String].imap(PersonName.apply)(_.value)
}
```

mapping によって、不正な domain 値が validation なしで生成されないかも確認する。

## ConnectionIO

複数の`ConnectionIO`は for 式で 1 つへ合成してから transact する。

```scala
val cio =
  for {
    _ <- cioa
    _ <- ciob
  } yield ()

transactor.transact(cio)
```

個別に transact して、意図した transaction 境界を分断しない。途中の失敗時に一部だけ commit されないか確認する。

## LogHandler

プロジェクトの Doobie version に対応した独自`LogHandler`を用意する。

- SQL 実行が error の場合は error level で記録する。
- それ以外は debug level で記録する。
- error log には trace ID、request ID、entity ID など、障害調査に必要な ID を含める。
- parameter、password、token、個人情報を出力しない。
- logging 自体の失敗で DB 処理の結果を変えない。
