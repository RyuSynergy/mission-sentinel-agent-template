---
applyTo: "detections/**/*.{yaml,yml,json,bicep}"
---

# 分析ルール (Analytics Rule) 作成規約

Sentinel の Scheduled Analytics Rule を作成する場合のフォーマット規約です。  
**新規作成や有効化は人間の承認が必須**(`copilot-instructions.md` §4.2)。

## 1. メタデータ必須項目

| フィールド | 必須 | 例 |
| --- | --- | --- |
| `displayName` | ✅ | `Suspicious PowerShell Encoded Command` |
| `description` | ✅ | 何を検知するか、ビジネス上のリスク |
| `severity` | ✅ | `Informational` / `Low` / `Medium` / `High` |
| `tactics` | ✅ | MITRE ATT&CK Tactic 名(複数可) |
| `techniques` | ✅ | `T1059.001` 形式 |
| `queryFrequency` | ✅ | ISO8601 (例: `PT1H`) |
| `queryPeriod` | ✅ | `queryFrequency` 以上 |
| `triggerOperator` / `triggerThreshold` | ✅ | `GreaterThan` / `0` |
| `entityMappings` | ✅ | Account / Host / IP など |
| `incidentConfiguration.createIncident` | ✅ | true 推奨 |
| `incidentConfiguration.groupingConfiguration` | ➖ | 同種アラートのグルーピング |

## 2. クエリ品質

- KQL 本体は [.github/instructions/kql.instructions.md](kql.instructions.md) に従う
- **必ず `entityMappings` でエンティティ抽出**できるカラムを `project` に含める
- 検知ロジックの根拠(なぜ怪しいか)を `description` に明記

## 3. ファイル配置

```
detections/
  <DataSource>/
    <RuleName>.yaml          # メインルール定義
    <RuleName>.tests.kql     # 検証用クエリ(陽性/陰性ケース)
    <RuleName>.md            # 詳細説明・チューニング履歴
```

## 4. レビュー観点(エージェントが提案する際の自己チェック)

- [ ] 既存ルールと重複していないか(`detections/` を grep)
- [ ] False Positive 想定を `description` または `.md` に書いたか
- [ ] `severity` は影響度に対して妥当か(過剰/過少でないか)
- [ ] エンティティが少なくとも 1 つ抽出されるか
- [ ] テスト用クエリで陽性ケースが取れるか確認したか

## 5. 雛形 (YAML)

```yaml
id: <GUID>
name: SuspiciousPowerShellEncodedCommand
displayName: Suspicious PowerShell Encoded Command
description: |
  Base64 エンコードされた PowerShell コマンドの実行を検知する。
  正規の管理スクリプトでも使用されるため、署名済みプロセスやサーバー管理セグメントは除外する。
severity: Medium
status: Available
requiredDataConnectors:
  - connectorId: MicrosoftThreatProtection
    dataTypes: [DeviceProcessEvents]
queryFrequency: PT1H
queryPeriod: PT1H
triggerOperator: GreaterThan
triggerThreshold: 0
tactics: [Execution, DefenseEvasion]
relevantTechniques: [T1059.001, T1027]
query: |
  DeviceProcessEvents
  | where TimeGenerated >= ago(1h)
  | where FileName =~ "powershell.exe"
  | where ProcessCommandLine has_any ("-enc", "-EncodedCommand")
  | project TimeGenerated, DeviceName, AccountName, ProcessCommandLine, InitiatingProcessFileName
entityMappings:
  - entityType: Account
    fieldMappings:
      - identifier: Name
        columnName: AccountName
  - entityType: Host
    fieldMappings:
      - identifier: HostName
        columnName: DeviceName
incidentConfiguration:
  createIncident: true
  groupingConfiguration:
    enabled: true
    reopenClosedIncident: false
    lookbackDuration: PT5H
    matchingMethod: Selected
    groupByEntities: [Host]
```
