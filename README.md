# my-agent-skills

codex 用の skills を管理する repository です。

skill-creator で作成していますが、人間が skill の中身をレビューするので多少性能が落ちても技術文書としての体裁を重視する方針です。

## setup

```sh
mise install
pnpm install
```

### Codexへのskill読み込み

repositoryのルートで次のコマンドを実行すると、repo固有のskillとしてCodexに読み込ませられます。

```sh
mkdir -p .agents/skills
ln -sfn ../../skills/commit-message .agents/skills/commit-message
ln -sfn ../../skills/git-add .agents/skills/git-add
ln -sfn ../../skills/functional-programming .agents/skills/functional-programming
```

反映するには、Codexで新しいセッションを開始します。

### textlintの実行

Markdown ファイルをチェックするには、次のコマンドを実行する。

```sh
pnpm lint
```

## 現在実装しているスキル

- `commit-message`
  日本語の Conventional Commits 形式で commit message を作成する。

- `git-add`
  変更内容を確認し、対象のファイルやを git add -p を利用し stage する。

- `functional-programming`
  コード生成とコードレビューで関数型のエッセンスを利用する。
