---
name: git-add
description: Gitの変更一覧とdiffを確認し、ユーザーが指定したファイルやhunkだけをstageするskill。git addを依頼されたときに使用する。git add -Aとgit add .は禁止し、git statusで対象を確認してからファイルを1つずつ指定する。新規ファイル以外は原則git add -pを使用し、100ファイル以上の変更がある場合だけユーザーの確認後にgit add -Aを許可する。
---

# Git Add

Git の変更を確認し、意図したファイルや hunk だけを stage する。stage だけを担当し、ユーザーから明示的に依頼されない限り commit や push は実行しない。

## 手順

1. `git status --short`で変更ファイルを確認する。
2. `git diff --stat`と`git diff -- <path>`で変更内容を確認する。untracked file は内容を確認してから対象にする。
3. stage する対象をユーザーの依頼と変更内容から決める。依頼に含まれないファイル、生成物、依存関係キャッシュ、秘密情報は stage しない。
4. 新規ファイルは、パスを 1 つだけ指定して`git add -- <path>`を実行する。
5. 変更済み・削除済みファイルは、原則として`git add -p -- <path>`で hunk 単位に確認しながら stage する。不要な hunk は stage しない。
6. stage 後に`git status --short`、`git diff --cached --name-status`、`git diff --cached --stat`で結果を確認する。

`git add -p`の対話では`y`、`n`、`s`、`q`だけを使用し、`e`（手動 patch 編集）は使用しない。hunk を適切に分割できない場合は stage を中断して、ユーザーに確認する。

## stageの単位

stage する単位は、1 つの目的を持つ 1 つの commit とする。ファイル単位ではなく、変更の目的と想定する Conventional Commits の prefix を基準に判断する。

- 1 つの機能や修正に必要な実装・test・docs は、複数の file type にまたがっていても同じ commit 単位にする。
- 無関係な documentation、独立した refactor、別の bug 修正などは別の commit 単位に分ける。
- 同じ file 内に複数の目的の変更がある場合は、`git add -p -- <path>`で hunk を分ける。
- 変更内容が複数の独立した prefix に該当する場合は、stage を止めて commit を分割するかユーザーに確認する。
- 1 つの commit 単位に含める file が複数ある場合も、各 file を個別のパスで stage する。

stage 前に「この stage 内容を 1 行の commit message で正確に表せるか」を確認する。表せない場合は、目的ごとの stage 案を提示してユーザーに確認する。

## 禁止事項

- `git add -A`を実行しない。
- `git add .`を実行しない。
- `git add --all`を実行しない。
- `git add *`など、対象を広く解釈する glob を使わない。
- `git add -u`で依頼されていない変更をまとめて stage しない。
- `git add -p`で`e`を選択して patch を手動編集しない。
- 変更内容を確認せずにファイルを stage しない。

ファイルを stage するときは、必ず`git status`で確認した具体的なパスを 1 つずつコマンドに指定する。

## 100ファイル以上の変更

`git status --short`で変更ファイル数を確認する。
100 個以上の場合は、全変更を stage するかユーザーに確認する。

ユーザーが明示的に許可した場合に限り、`git add -A`を実行してよい。許可がない場合は、対象ファイルを絞って 1 つずつ stage する。100 ファイル未満では、いかなる場合も`git add -A`を使わない。

## 完了条件

stage 後の`git diff --cached`を確認する。
ユーザーの意図しないファイルや hunk があれば、stage を止めて確認する。
