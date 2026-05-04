---
mode: agent
description: ユーザーまたはデバイスのリスクスコアを多軸で算出
---

# エンティティリスクスコアリング

指定されたユーザーまたはデバイスを多軸で評価し、リスクスコアと根拠を提示します。

## 入力

- **エンティティ種別**: ${input:entityType:user または device}
- **識別子**: ${input:entityId:UPN またはデバイス名}
- **評価期間**: ${input:lookback:既定 30d}

## 評価軸

### User の場合
| 軸 | データソース | スコア配点 |
| --- | --- | --- |
| Identity Protection リスク | `AADUserRiskEvents`, `AADRiskyUsers` | 0–25 |
| 異常サインイン(地理 / IP / アプリ) | `SigninLogs` | 0–20 |
| MFA 失敗 / 総当たり兆候 | `SigninLogs` (ResultType) | 0–15 |
| 関連インシデント数 | `SecurityIncident` | 0–20 |
| 特権ロール保有 | `IdentityInfo` / `RoleManagementPolicyAssignment` | 0–10 |
| 受信フィッシングメール数 | `EmailEvents` (ThreatTypes) | 0–10 |

### Device の場合
| 軸 | データソース | スコア配点 |
| --- | --- | --- |
| MDE Device Risk | `DeviceInfo` (RiskScore) | 0–25 |
| 高 Severity アラート数 | `DeviceAlertEvents` | 0–20 |
| 未パッチ脆弱性 | `DeviceTvmSoftwareVulnerabilities` | 0–20 |
| 異常プロセス実行 | `DeviceProcessEvents` (rare process) | 0–15 |
| 外部通信異常(beaconing) | `DeviceNetworkEvents` | 0–10 |
| Onboarding / EDR 健全性 | `DeviceInfo` (OnboardingStatus, SensorHealthState) | 0–10 |

## レーティング

- 0–24: Low
- 25–49: Medium
- 50–74: High
- 75–100: Critical

## アウトプット

- `reports/risk/<entity-type>_<id-slug>_YYYY-MM-DD.md` に保存
- 構造:
  - 総合スコア / レーティング
  - 軸別スコアと根拠クエリ
  - 推奨アクション(隔離 / 強制リセット / 監視強化 など)
  - **対応アクションの実行は人間承認必須**
