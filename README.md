# MISSION-Sentinel

セキュリティオペレーション(SecOps)のための VSCode + AI エージェント作業環境。

## 対象スコープ

- Microsoft Sentinel
- Microsoft Defender XDR (Endpoint / Identity / Office / Cloud Apps)
- Microsoft Entra ID (サインイン / 監査ログ)
- Azure リソース (Activity Log / Resource Graph)

## ディレクトリ構成

```
.
├── .github/
│   ├── copilot-instructions.md      # リポジトリ全体の指示(全エージェント共通)
│   ├── instructions/                # ファイル種別ごとの作成規約 (applyTo)
│   ├── prompts/                     # 定型プロセスの再利用プロンプト
│   └── chatmodes/                   # VSCode カスタムチャットモード
├── skills/                          # ドメイン知識スキル (SKILL.md)
├── detections/                      # 分析ルール (KQL / ARM / Bicep / YAML)
├── hunts/                           # ハンティングクエリ (.kql)
├── runbooks/                        # 手順書 (Markdown)
├── reports/                         # 生成レポートとテンプレート
│   └── templates/
├── agents/                          # 外部実行エージェントの定義
│   ├── copilot-studio/              # Microsoft Copilot Studio エージェント
│   └── security-copilot/            # Microsoft Security Copilot エージェント
└── Project-Outline.md
```

## ロール別の入口

| やりたいこと | 開く場所 |
| --- | --- |
| 怪しいユーザーを調査したい(あいまい依頼) | [.github/prompts/investigate-suspicious-user.prompt.md](.github/prompts/investigate-suspicious-user.prompt.md) |
| インシデントを初動トリアージしたい | [.github/prompts/triage-incident.prompt.md](.github/prompts/triage-incident.prompt.md) |
| IOC をハンティングしたい | [.github/prompts/hunt-ioc.prompt.md](.github/prompts/hunt-ioc.prompt.md) |
| 週次サマリを作りたい | [.github/prompts/weekly-report.prompt.md](.github/prompts/weekly-report.prompt.md) |
| 検知ルールを作る/チューニングしたい | [.github/prompts/tune-detection.prompt.md](.github/prompts/tune-detection.prompt.md) |
| 脆弱性アラートを評価したい | [.github/prompts/vuln-risk-assess.prompt.md](.github/prompts/vuln-risk-assess.prompt.md) |
| エンティティのリスクを評価したい | [.github/prompts/risk-score-entity.prompt.md](.github/prompts/risk-score-entity.prompt.md) |

## 言語ポリシー

- **コミットメッセージ・レポート・コメント**: 日本語
- **コード・KQL・識別子・ファイル名・タグ・技術用語**: 英語

## 安全レベル

このリポジトリのエージェントは以下を許可します。

- 読み取り(クエリ実行、ログ取得)
- 低リスクな書き込み(インシデントへのコメント追加、タグ付け、レポートファイル生成)

以下は**人間の承認が必要**です。

- 分析ルール / オートメーションルールの本番適用
- インシデントの Close / Resolve
- ユーザー / デバイスのブロック等の対応アクション

詳細は [.github/copilot-instructions.md](.github/copilot-instructions.md) を参照。
