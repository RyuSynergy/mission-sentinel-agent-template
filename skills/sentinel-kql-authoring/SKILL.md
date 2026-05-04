# SKILL: Sentinel KQL Authoring

**いつ使うか**: Sentinel / Defender XDR テーブルへの KQL を新規作成・最適化するとき。

## 0. このワークスペースで実在するテーブル

**最初に必ず確認**: [workspace-tables.md](workspace-tables.md) — このワークスペースで実際に取り込まれているテーブルとカスタムログのカタログ。  
ここに無いテーブルを使う前に `mcp_microsoft_sen_search_tables` でスキーマと存在を確認すること。

## 主要テーブル早見表(Microsoft 標準スキーマ参考)

### Microsoft Sentinel コア
| テーブル | 用途 |
| --- | --- |
| `SecurityIncident` | インシデントメタデータ |
| `SecurityAlert` | アラート |
| `ThreatIntelligenceIndicator` | TI フィード IOC |
| `Heartbeat` | エージェント死活 |

### Entra ID
| テーブル | 用途 |
| --- | --- |
| `SigninLogs` | 対話型サインイン |
| `AADNonInteractiveUserSignInLogs` | 非対話サインイン |
| `AADServicePrincipalSignInLogs` | SPN サインイン |
| `AuditLogs` | ディレクトリ変更 |
| `AADUserRiskEvents`, `AADRiskyUsers` | Identity Protection |

### Defender for Endpoint (Advanced Hunting)
| テーブル | 用途 |
| --- | --- |
| `DeviceInfo` | デバイス基礎情報 |
| `DeviceProcessEvents` | プロセス実行 |
| `DeviceNetworkEvents` | ネットワーク通信 |
| `DeviceFileEvents` | ファイル操作 |
| `DeviceLogonEvents` | ログオン |
| `DeviceImageLoadEvents` | DLL ロード |
| `DeviceRegistryEvents` | レジストリ |
| `DeviceTvmSoftwareInventory` | インストールソフト |
| `DeviceTvmSoftwareVulnerabilities` | 脆弱性 |

### Defender for Office 365
| テーブル | 用途 |
| --- | --- |
| `EmailEvents` | メール配信メタデータ |
| `EmailAttachmentInfo` | 添付情報 |
| `EmailUrlInfo` | メール内 URL |
| `UrlClickEvents` | Safe Links クリック |

### Defender for Identity / Cloud Apps
| テーブル | 用途 |
| --- | --- |
| `IdentityLogonEvents` | AD / AAD 認証 |
| `IdentityQueryEvents` | AD クエリ |
| `IdentityDirectoryEvents` | AD 変更 |
| `CloudAppEvents` | MCAS / DCA イベント |

### Azure
| テーブル | 用途 |
| --- | --- |
| `AzureActivity` | サブスクリプション操作 |
| `AzureDiagnostics` | リソース診断 |

## 実装パターン

### ベースライン逸脱(rare event)
```kusto
let lookback = 30d;
let recent = 1d;
DeviceProcessEvents
| where TimeGenerated >= ago(lookback)
| summarize Cnt=count() by FileName, DeviceName
| where Cnt < 3   // ベースライン
| join kind=inner (
    DeviceProcessEvents
    | where TimeGenerated >= ago(recent)
) on FileName, DeviceName
```

### Beaconing 検知
```kusto
DeviceNetworkEvents
| where TimeGenerated >= ago(1d)
| where ActionType == "ConnectionSuccess"
| summarize Count=count(), Intervals=make_list(bin(TimeGenerated, 1m))
    by DeviceName, RemoteIP
| where Count > 30
| extend StdDev = todouble(array_length(Intervals))
```

### Impossible Travel
```kusto
SigninLogs
| where TimeGenerated >= ago(1d)
| where ResultType == 0
| project TimeGenerated, UserPrincipalName, IPAddress, City=tostring(LocationDetails.city)
| sort by UserPrincipalName, TimeGenerated asc
| serialize
| extend PrevCity = prev(City), PrevTime = prev(TimeGenerated), PrevUser = prev(UserPrincipalName)
| where UserPrincipalName == PrevUser and City != PrevCity
| extend MinutesDiff = datetime_diff('minute', TimeGenerated, PrevTime)
| where MinutesDiff < 60
```

## アンチパターン

| ❌ NG | ✅ OK |
| --- | --- |
| `search "192.0.2.10"` | テーブル指定して `where IPAddress == "192.0.2.10"` |
| `where Field contains "x"` | `where Field has "x"`(可能な限り) |
| `summarize` 後に `where` で絞る | `where` で先に絞る |
| 巨大テーブル同士の `join` | 小さい側を左、`hint.strategy=broadcast` 検討 |

## 参照

- 公式リファレンス: `microsoft_docs_search` で `KQL operator <name>` を検索
- スキーマ確認: `mcp_microsoft_sen_search_tables` でテーブル名を検索
