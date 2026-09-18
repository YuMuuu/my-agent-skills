# youtube-knowledge

YouTube 動画を参考資料として使い、ユーザの依頼に答える skill です。
動画と字幕を取得し、必要に応じて静止画や短い区間を確認します。

## 使用方法

YouTube 動画を参考にするよう明示的に依頼するときに使用します。
暗黙起動は無効にしているため、Codex では `$youtube-knowledge` で呼び出します。

```text
$youtube-knowledge このYouTube動画を参考に、説明されている設定手順をまとめて。
```

実行手順と判断基準は [SKILL.md](SKILL.md) に記載しています。
起動設定は [agents/openai.yaml](agents/openai.yaml) で管理しています。

## 使用ツール

- `yt-dlp`: 動画とメタデータを取得します。
- `youtube-transcript-api`: 既存の字幕を時刻付きで取得します。
- `ffmpeg`: 必要な場面を静止画や短い区間として取り出します。
- `ffprobe`: 取得した動画ファイルを確認します。