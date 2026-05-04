# Workspace Tables (実在テーブルカタログ)

ワークスペース `log-sentineldatalake` (`<workspace-id>`) で **実際に取り込み実績がある** テーブル一覧。  
**Last verified: 2026-05-04** / 参照期間: 直近 30 日 / 出典: `union withsource=TableName * | summarize ...`

> エージェントへの指示: 調査でログを探すときは、**まずこの一覧から候補を選ぶ**。ここに無いテーブルを参照したい場合は `mcp_microsoft_sen_search_tables` でスキーマを確認してから使うこと。

## 1. カテゴリ別サマリ

### 1.1 Microsoft Defender for Endpoint (MDE)
| テーブル | 直近 30d 件数 | 主な用途 |
| --- | --- | --- |
| `DeviceFileEvents` | 53,110 | ファイル作成/変更/削除。マルウェア投下、永続化、データ持ち出しの起点 |
| `DeviceRegistryEvents` | 39,131 | レジストリ操作。永続化(Run キー等)、設定改ざん |
| `DeviceEvents` | 12,573 | MDE が発する**雑多なイベント**(ASR、AMSI、Tamper、Defender アクション 等) |
| `DeviceNetworkInfo` | 9,228 | NIC・IP・MAC・接続中ネットワーク。資産情報スナップショット |
| `DeviceNetworkEvents` | 9,210 | アウトバウンド/インバウンド通信。C2、ビーコン、横展開 |
| `DeviceProcessEvents` | 8,868 | プロセス起動。実行系の主要証跡 |
| `DeviceFileCertificateInfo` | 5,178 | 実行ファイル署名情報。署名なし/失効/不明な発行者の検出 |
| `DeviceImageLoadEvents` | 2,693 | DLL ロード。プロセスインジェクション、LOLBin 経由ロード |
| `DeviceLogonEvents` | 2,081 | ローカル/RDP/ネットワークログオン。横展開、特権利用 |
| `DeviceInfo` | 971 | デバイス基礎情報スナップショット(OS、Risk、Onboarding 状況) |

### 1.2 Microsoft Defender for Identity (MDI)
| テーブル | 直近 30d 件数 | 主な用途 |
| --- | --- | --- |
| `IdentityLogonEvents` | 1,894 | AD/AAD 認証。Kerberos/NTLM、対話型/非対話型 |
| `IdentityQueryEvents` | 546 | LDAP/SAMR クエリ。偵察(Recon)検知 |
| `IdentityDirectoryEvents` | 15 | AD オブジェクトの変更(GPO、グループメンバ、属性) |

### 1.3 Microsoft Defender for Office 365 (MDO)
| テーブル | 直近 30d 件数 | 主な用途 |
| --- | --- | --- |
| `EmailUrlInfo` | 333 | メール内 URL。フィッシング URL 解析 |
| `EmailEvents` | 40 | メール配信メタデータ。送信者/受信者/件名/Threats |
| `EmailAttachmentInfo` | 26 | 添付ファイル情報。ハッシュ、種別、Threats |

### 1.4 Microsoft Defender XDR 共通(統合スキーマ)
| テーブル | 直近 30d 件数 | 主な用途 |
| --- | --- | --- |
| `AlertEvidence` | 1,618 | アラートに紐づくエンティティ証拠(File/Process/IP/User 等) |
| `AlertInfo` | 66 | アラートのメタデータ(Title, Severity, Category, ServiceSource) |

### 1.5 Microsoft Sentinel
| テーブル | 直近 30d 件数 | 主な用途 |
| --- | --- | --- |
| `ThreatIntelIndicators` | 440,610 | TI フィードの IOC(IP/Domain/URL/Hash) |
| `SecurityAlert` | 209 | Sentinel に集約されたアラート |
| `SecurityIncident` | 50 | インシデント(複数アラートのグループ) |
| `SentinelHealth` | 514 | データコネクタ健全性、取り込みエラー |
| `SentinelAudit` | 22 | Sentinel 構成変更の監査 |

### 1.6 Microsoft Entra ID
| テーブル | 直近 30d 件数 | 主な用途 |
| --- | --- | --- |
| `AuditLogs` | 5,523 | ディレクトリ変更(ユーザー/グループ/ロール/アプリ) |
| `SigninLogs` | 199 | 対話型サインイン |

### 1.7 Azure リソース
| テーブル | 直近 30d 件数 | 主な用途 |
| --- | --- | --- |
| `AzureActivity` | 557 | サブスクリプション操作(管理プレーン) |
| `Operation` | 19 | LA ワークスペース内オペレーション |
| `Usage` | 3,957 | データ取り込み量(コスト/コネクタ確認) |

### 1.8 カスタムログ (`_CL`)
| テーブル | 直近 30d 件数 | 主な用途 |
| --- | --- | --- |
| `GatewayTraffic_CL` | 6,312 | ファイアウォール / プロキシのトラフィックログ(CEF 形式) |
| `VPNAuthEvents_CL` | 3,000 | VPN 認証イベント(CEF 形式) |

---

## 2. 注意事項

- **このワークスペースには `AADNonInteractiveUserSignInLogs` / `AADServicePrincipalSignInLogs` は取り込まれていない**(2026-05-04 時点)。SPN サインインを調査したい場合は事前にユーザーへ確認
- **MDE Advanced Hunting テーブル `DeviceTvmSoftwareInventory` / `DeviceTvmSoftwareVulnerabilities` も未取り込み**。脆弱性評価は MDVM 側で実施するか、追加コネクタの提案
- **`CloudAppEvents` (MDA) / `OfficeActivity` も未取り込み**。SaaS 操作監査が必要なら別途接続

---

## 3. カスタムログのスキーマ詳細

### 3.1 `GatewayTraffic_CL`
| カラム | 型 |
| --- | --- |
| `TenantId` | guid |
| `TimeGenerated` | datetime |
| `DeviceVendor` / `DeviceProduct` / `DeviceVersion` | string |
| `DeviceEventClassID` | string |
| `Activity` | string |
| `LogSeverity` | int |
| `SourceIP` / `DestinationIP` | string |
| `DestinationPort` | int |
| `SentBytes` / `ReceivedBytes` | long |
| `Protocol` | string |
| `DeviceAction` | string (`allow`/`deny` 等) |
| `Message` | string |

> CEF 由来のスキーマ。アウトバウンドのトラフィック分析、ブロック/許可の傾向、未知ポートへの接続検知等に使う。

### 3.2 `VPNAuthEvents_CL`
| カラム | 型 |
| --- | --- |
| `TenantId` | guid |
| `TimeGenerated` | datetime |
| `DeviceVendor` / `DeviceProduct` / `DeviceVersion` | string |
| `DeviceEventClassID` | string |
| `Activity` | string |
| `LogSeverity` | int |
| `SourceIP` | string |
| `SourceUserName` | string |
| `EventOutcome` | string (`success`/`failure` 等) |
| `DeviceAction` | string |
| `Message` | string |

> VPN 認証成功/失敗の追跡、外部からのブルートフォース、業務時間外接続の検知に使う。

---

## 4. 標準テーブルのスキーマ確認方法

標準 Microsoft テーブル(`SigninLogs`, `DeviceProcessEvents` 等)のスキーマは Microsoft Learn が一次情報。

```
microsoft_docs_search "<TableName> schema"
```

または MCP で直接確認:

```kusto
<TableName> | getschema | project ColumnName, ColumnType
```

---

## 5. 更新手順

このカタログは月 1 回、または以下のタイミングで更新する:
- 新規データコネクタ接続時
- カスタムログ (`_CL`) の新規追加時
- データソースの停止 / 削除時

更新コマンド(エージェントが実行可):

```kusto
union withsource=TableName *
| where TimeGenerated > ago(30d)
| summarize Count=count(), LastSeen=max(TimeGenerated), FirstSeen=min(TimeGenerated) by TableName
| order by Count desc
```
