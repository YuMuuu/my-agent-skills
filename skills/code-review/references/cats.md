# CatsとCats Effect

Cats または Cats Effect を利用する Scala コードで適用する。

## interfaceと内部実装

Cats Effect `IO`は interface へ公開してよい。想定可能な失敗を扱うレイヤー境界では、次のように effect の内側へ`Either`を置く。

```scala
def find(personId: PersonId): IO[Either[RepositoryError, Person]]
```

`EitherT`や`OptionT`は内部合成に利用し、interface へ公開しない。

```scala
// 避ける
def find(personId: PersonId): EitherT[IO, RepositoryError, Person]
```

## EitherT

2 つ以上の`Effect[Either[E, A]]`を逐次合成する場合だけ`EitherT`を利用する。`Effect`は`Future`または`IO`など、プロジェクトで採用した effect を表す。

```scala
def register(
    command: RegisterPerson
): IO[Either[RegistrationError, Person]] =
  (
    for {
      person <- EitherT(validate(command))
      savedPerson <- EitherT(save(person))
      _ <- EitherT(publish(PersonRegistered(savedPerson.id)))
    } yield savedPerson
  ).value
```

単一の`Effect[Either[E, A]]`を包んですぐ`.value`するだけの`EitherT`は利用しない。

## OptionT

2 つ以上の`Effect[Option[A]]`を逐次合成する場合だけ`OptionT`を利用する。

```scala
(
  for {
    person <- OptionT(findPerson(personId))
    company <- OptionT(findCompany(person.companyId))
  } yield company
).value
```

単一の`Effect[Option[A]]`を包んですぐ`.value`するだけの`OptionT`は利用しない。

`Either`と`Option`が混在する場合は、処理全体の失敗表現を決める。不要に transformer を積み重ねず、`fromOptionF`などで主要な transformer へ変換する。

## 構文

`Either`の左右を変換する場合は、次の構文を利用する。

- `bimap`
- `leftMap`

effect の成功値を置き換える場合は`as`を利用する。成功値を破棄して`Unit`にする場合は`void`を利用する。`IO`や Doobie の`ConnectionIO`など、Cats の構文を利用できる effect が対象になる。

```scala
savePerson(person).as(person.id)
```

```scala
updatePerson(person).void
```

この規約は単一の effect の結果を変換する場合に適用する。複数の effect の実行順序は for 式で表現する。

評価順序だけを記号で表す次の演算子は原則として利用せず、for 式で処理順序を明示する。

- `*>`
- `<*`
- `>>`

```scala
for {
  _ <- savePerson(person)
  result <- publishEvent(event)
} yield result
```
