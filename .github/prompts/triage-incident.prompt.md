---
mode: agent
description: Sentinel インシデントを初動トリアージし、所見と推奨アクションをまとめる
---

# インシデント初動トリアージ

あなたは **Tier1 SOC アナリスト** として、指定された Sentinel インシデントを初動トリアージしてください。

## 入力

- **Incident ID**: ${input:incidentId:インシデントIDを入力(Defender XDR 番号を優先)}
- **対象ワークスペース**: ${input:workspace:Sentinel ワークスペース名(空欄で既定)}

入力が無い場合は、ユーザーに必ず確認すること。

> 番号体系の注意: ユーザーが渡す ID は通常 **Defender XDR の番号(`ProviderIncidentId`)**。Sentinel API は内部の `IncidentNumber` を要求するため、まず両者をマッピングする。

## 実行手順

1. **基礎情報の取得 + 番号マッピング**
   - `mcp_microsoft_sen_query_lake` で `SecurityIncident` から当該インシデントを取得
   - **入力 ID は Defender XDR 番号として `ProviderIncidentId` で先に検索**。ヒットしなければ `IncidentNumber` で再検索
   - 取得後は必ず `XdrIncidentId` と `SentinelIncidentNumber` の両方を控える
   - クエリ例:
     ```kusto
     SecurityIncident
     | where TimeGenerated >= ago(90d)
     | where ProviderIncidentId == "<入力ID>" or IncidentNumber == toint("<入力ID>")
     | summarize arg_max(TimeGenerated, *) by IncidentNumber
     | project XdrIncidentId=ProviderIncidentId, SentinelIncidentNumber=IncidentNumber,
               Title, Severity, Status, Owner=tostring(Owner.assignedTo), AlertIds, Entities
     ```
   - 以降のレポートでは `XDR #<id> (Sentinel #<num>)` 形式で参照
2. **関連アラートの取得**
   - `SecurityAlert` から `AlertIds` に該当するレコードを取得
   - 各アラートの `Tactics` / `Techniques` / `Description` を確認
3. **エンティティのエンリッチメント**
   - User: `SigninLogs` の過去 7 日のサインイン傾向(地理・アプリ・成功/失敗)
   - Device: `DeviceInfo` / `DeviceEvents` で OS / オンボーディング状況 / 最近の高重大度イベント
   - IP: `ThreatIntelligenceIndicator` に該当するか / 過去 30 日の出現頻度
   - Hash / Domain: 同上
4. **横展開の確認**
   - 同じユーザー/デバイス/IP に紐づく**他のオープンインシデント**を検索
   - 過去 30 日の類似アラート(同じ DetectionType)
5. **評価**
   - True Positive / Benign Positive / False Positive / Inconclusive を判定
   - 確度: High / Medium / Low と根拠を 3 行以内で
6. **MITRE ATT&CK マッピング**(`Txxxx Technique Name` 形式)
7. **推奨アクション**(即時 / 中期 / 検知改善)
8. **レポート出力**
   - `reports/incidents/YYYY-MM-DD_<incidentId>_<slug>.md` に保存
   - フォーマットは [.github/instructions/incident-report.instructions.md](../instructions/incident-report.instructions.md) に従う
9. **インシデントへのコメント追加**(承認済みの低リスク書き込み)
   - 「自動トリアージにより評価: <TP/BP/FP> / 確度: <H/M/L>」を 1 行で記載
   - 詳細は本リポジトリのレポートパスを参照させる

## 守ること

- クエリは必ず **時間範囲を明示**(`copilot-instructions.md` §5)
- 機密情報(メール本文・トークン)は**マスク**
- インシデントの **Status / Severity / Owner は変更しない**(承認必須)
- 不明点は推測せずユーザーに質問する

## アウトプット

最終的に Markdown レポートのパスと、評価サマリ(3 行)を返してください。
