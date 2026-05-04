# Security Copilot エージェント / Promptbook

Microsoft Security Copilot で実行する Promptbook / カスタムエージェントの置き場。

## 推奨される最初の Promptbook

| 名前 | 内容 |
| --- | --- |
| `IncidentDeepDive` | Sentinel インシデントを多製品横断で深堀り(Defender / Entra / Intune) |
| `IdentityRiskReview` | Entra Risky Users を一括レビュー |
| `VulnTriage` | MDVM 推奨事項のリスク評価 |

## ファイル構成(推奨)

```
security-copilot/
  promptbooks/
    <name>.md           # プロンプト連鎖の定義
  custom-plugins/
    <name>/
      manifest.json
      openapi.yaml
  README.md
```

## 設計時のチェック

- [ ] 各プロンプトは独立して再実行可能
- [ ] 出力はマークダウンで構造化
- [ ] 機密データはマスク
- [ ] **対応アクション(無効化・隔離等)を Promptbook に組み込まない**(別プロセスで人間承認)
