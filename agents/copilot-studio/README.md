# Copilot Studio エージェント

VSCode で安定運用できるようになった定型プロセスを Copilot Studio エージェントとして定義する場所。

## 推奨される最初のエージェント

| エージェント名 | 役割 | トリガー候補 |
| --- | --- | --- |
| `Sentinel-Triage-Bot` | インシデントトリアージとレポート生成 | Sentinel インシデント作成 webhook |
| `Weekly-SecOps-Reporter` | 週次サマリ生成 | スケジュール(月曜 09:00 JST) |
| `IOC-Hunter` | Teams で IOC を投稿すると横断検索 | Teams メッセージ |

## ファイル構成(推奨)

```
copilot-studio/
  <agent-name>/
    agent.yaml          # エージェント名・説明・公開設定
    instructions.md     # システムプロンプト(本リポジトリの copilot-instructions の SecOps 部分を抽出)
    topics/             # トピック定義
    actions/            # コネクタ / Power Automate / カスタムコネクタ呼び出し
    knowledge/          # ナレッジソース(URL / SharePoint / ファイル)
    README.md
```

## 設計時のチェック

- [ ] 指示の冒頭に「**安全境界**」を必ず置く(本番変更禁止 / Close 禁止 等)
- [ ] アクションは**最小権限**(Read 推奨、Write は用途限定)
- [ ] エラー時のフォールバック(人間にエスカレ)
- [ ] 監査用に実行ログをワークスペースへ書き出す
