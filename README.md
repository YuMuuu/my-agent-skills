# my-agent-skills

codex 用の skills を管理する repository です。

skill-creator で作成しているが、人間が skill の中身をレビューするので多少性能が落ちても技術文書としての体裁を重視する。

## setup

```sh
mise install
pnpm install
```

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
