---
name: commit-message
description: stagedされたGit変更を確認し、日本語で簡潔なConventional Commits形式のcommit messageを作成するskill。commit messageの作成を依頼されたときや、変更内容に適したprefixの分類を求められたときに使用する。異なるprefixに該当する独立した変更が含まれる場合は、commitを分割するか確認する。
---

# Commit Message

現在 staged されている変更から Git の commit message を作成する。message は日本語で記述する。この skill は message の作成だけを行い、ユーザーから明示的に依頼されない限り`git commit`は実行しない。

## 手順

1. staged の状態を確認する。
   `git status --short`と`git diff --cached --stat`を使う。
2. `git diff --cached`で staged された patch を読む。message の根拠にするのは staged された変更だけとし、unstaged や untracked のファイルは含めない。
3. 変更を許可された prefix のいずれか 1 つに分類する。明確に独立した種類の変更が 2 つ以上含まれている場合は、最終的な message を作成する前に commit を分割するかユーザーに確認する。混在した patch に対して勝手に 1 つの prefix を選ばない。
4. 分割に同意された場合は、commit ごとの変更グループと message 案を提示する。ただし、明示的な指示なしに stage・unstage・commit を実行しない。
5. 分割しない場合は、指定された行数で最適な message を出力する。

staged された変更がない場合は、要約できる staged diff がないことを伝え、先に`git add`を実行するよう案内する。working tree の diff で代用しない。

## messageのルール

- デフォルトは必ず 1 行にする：`type: 簡潔な変更内容`。
- ユーザーが 2 行 commit を明示的に要求した場合だけ、必ず 2 行にする。1 行目は Conventional Commits の header、2 行目は背景・動作・影響の簡潔な説明とする。行間に空行を入れない。
- 3 行の commit message は絶対に作成しない。詳細を書ききれない場合は、3 行目を追加せず省略する。
- Conventional Commits の構文に従う。scope が有用な場合は`type(scope): description`とし、破壊的変更は`!`で示せる。例：`feat(api)!: 旧endpointを削除`。
- description は具体的かつ簡潔にし、命令形で書く。末尾にピリオドを付けない。
- ユーザーが求めない限り、message の前後に箇条書き・引用符・Markdown code fence・issue trailer・説明文を付けない。
- staged された patch から確認できない変更を記述しない。

## 使用できるprefix

次の中でもっとも適切で具体的な prefix を選ぶ。

- `feat`: ユーザーから見える機能を追加する
- `fix`: bug や誤った動作を修正する
- `docs`: ドキュメントだけを変更する
- `test`: test を追加・変更する
- `refactor`: 動作を変えずに構造を変更する
- `perf`: performance を改善する
- `style`: 動作に影響しない formatting や whitespace を変更する
- `build`: build system や build に影響する依存関係を変更する
- `ci`: CI 設定や workflow を変更する
- `chore`: 他の prefix に該当しない保守作業を行う
- `revert`: 過去の commit を取り消す

## 複数の変更が混在する場合

feature と無関係な documentation は混在した patch と判断する。
bug 修正と独立した refactor も同様に扱う。
独立した file や hunk が異なる目的を持つ場合も同様に扱う。
1 つのまとまった変更が実装・test・docs にまたがる場合は、file type が複数でも 1 つの prefix でよい。
file type が複数あるだけで分割を求めない。

分割が適切そうな場合は、次のように確認する。
`stagedされた変更にfeatとdocsの内容が含まれているようです。commitを分割しますか？`
ユーザーの回答を待ってから message を確定する。
