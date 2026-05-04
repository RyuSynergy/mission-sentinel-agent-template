---
applyTo: "reports/**/*.md"
---

# インシデントレポート作成規約

`reports/` 配下のインシデント / ハンティングレポートはこの規約に従ってください。

## 1. ファイル名

```
reports/incidents/YYYY-MM-DD_<incidentId>_<short-slug>.md
reports/hunts/YYYY-MM-DD_<hunt-name>.md
reports/weekly/YYYY-WW.md
```

## 2. 必須セクション

```markdown
# <タイトル>

- XDR Incident ID: `<ProviderIncidentId>` (主、該当する場合)
- Sentinel Incident ID: `<IncidentNumber>` (補助、該当する場合)
- 発生日時: YYYY-MM-DD HH:MM (JST)
- 報告者: <name / agent>
- ステータス: Investigating / Contained / Resolved / Monitoring
- 確度: High / Medium / Low

## 概要 (Summary)

## タイムライン (Timeline)
| 時刻 (JST) | 出来事 | 出典 |
| --- | --- | --- |

## 影響範囲 (Scope)
- 影響を受けたユーザー / デバイス / リソースをバックティック付きで列挙

## 観測された事実 (Observations)

## 評価 (Assessment)
- TP / BP / FP / Inconclusive
- 根拠

## MITRE ATT&CK マッピング

## 推奨アクション (Recommendations)
- 即時対応
- 中期対策
- 検知改善案

## 使用したクエリ (Queries Used)
\`\`\`kusto
// 実行した KQL を貼り付け
\`\`\`

## 添付 / 関連リンク
```

## 3. 機密情報の取り扱い

- メール本文・ファイル内容など**ペイロードはマスク**する(先頭 30 文字 + `...[truncated]`)
- パスワード・トークン・セッション ID は**絶対に貼らない**
- 顧客 PII は**ID 化**(例: `User-001`)し、対応表は別ファイル(コミット対象外)で管理

## 4. インシデント番号の表記規約(重要)

- **主に使う番号は Defender XDR のインシデント番号**(Sentinel `SecurityIncident.ProviderIncidentId` / Defender ポータル URL `incidents/<id>` と同じ)
- Sentinel ネイティブの `IncidentNumber` は**補助番号**として併記する(Sentinel UI / API での参照用)
- 表記例:
  - 単体表記: `XDR #104`
  - 併記: `XDR #104 (Sentinel #12)`
- マッピング取得クエリ:
  ```kusto
  SecurityIncident
  | where TimeGenerated >= ago(30d)
  | summarize arg_max(TimeGenerated, *) by IncidentNumber
  | where ProviderName == "Microsoft XDR"
  | project XdrIncidentId = ProviderIncidentId, SentinelIncidentNumber = IncidentNumber, Title, Severity, Status
  ```
- ファイル名にも XDR 番号を使う: `reports/incidents/YYYY-MM-DD_xdr-<ProviderIncidentId>_<slug>.md`
- `ProviderName != "Microsoft XDR"` のインシデント(他コネクタ由来)は Sentinel `IncidentNumber` を主にし、その旨を明記する

## 5. エンティティ表記

- IP アドレス: `192.0.2.10`
- ハッシュ: `SHA256: abcdef...`
- UPN: `user@example.com`
- デバイス名: `HOST-12345`

すべてバックティック付き。翻訳・整形しない。
