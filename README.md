# my-agent-skills

codex 用の skills を管理する repository です。

skill-creator で作成していますが、人間が skill の中身をレビューするので多少性能が落ちても技術文書としての体裁を重視する方針です。

## setup

```sh
mise install
pnpm install
```

### Codex全体へのskill読み込み

repository のルートで次のコマンドを実行します。Codex 全体で利用する skill として読み込ませられます。

```sh
CODEX_SKILLS_DIR="${CODEX_HOME:-$HOME/.codex}/skills"
mkdir -p "$CODEX_SKILLS_DIR"
ln -sfn "$PWD/skills/commit-message" "$CODEX_SKILLS_DIR/commit-message"
ln -sfn "$PWD/skills/code-comments" "$CODEX_SKILLS_DIR/code-comments"
ln -sfn "$PWD/skills/git-add" "$CODEX_SKILLS_DIR/git-add"
ln -sfn "$PWD/skills/functional-programming" "$CODEX_SKILLS_DIR/functional-programming"
```

反映するには、Codex で新しいセッションを開始します。

### textlintの実行

Markdown ファイルをチェックするには、次のコマンドを実行する。

```sh
pnpm lint
```

## 現在実装しているスキル

- `code-comments`
  日本語の通常コメント、TODO、FIXME を作成・改善する。処理内容（what）の繰り返しを避け、コードから分からない理由や制約を説明する。

- `commit-message`
  日本語の Conventional Commits 形式で commit message を作成する。

- `git-add`
  変更内容を確認し、対象のファイルやを git add -p を利用し stage する。

- `functional-programming`
  コード生成とコードレビューで関数型のエッセンスを利用する。
