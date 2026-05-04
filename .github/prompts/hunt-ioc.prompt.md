---
mode: agent
description: IOC(IP / Hash / Domain / URL / UPN)を主要テーブル横断でハンティング
---

# IOC ハンティング

指定された IOC を Sentinel / Defender XDR の主要テーブル横断で検索し、出現状況をまとめます。

## 入力

- **IOC**: ${input:ioc:検索対象の IOC(IP/Hash/Domain/URL/UPN)を入力}
- **時間範囲**: ${input:lookback:振り返り期間 例: 30d (既定 30d)}

## 実行手順

1. **IOC タイプ判定**(IP / SHA256 / SHA1 / MD5 / Domain / URL / UPN / Email)
2. **対象テーブルの選定**(タイプ別)
   | IOC | 主テーブル |
   | --- | --- |
   | IP | `SigninLogs`, `AADNonInteractiveUserSignInLogs`, `DeviceNetworkEvents`, `CommonSecurityLog`, `AzureActivity` |
   | Hash | `DeviceFileEvents`, `DeviceProcessEvents`, `DeviceImageLoadEvents`, `EmailAttachmentInfo` |
   | Domain/URL | `DeviceNetworkEvents`, `EmailUrlInfo`, `UrlClickEvents`, `DnsEvents` |
   | UPN/Email | `SigninLogs`, `EmailEvents`, `IdentityLogonEvents`, `OfficeActivity` |
3. **クエリ生成と実行**
   - 各テーブルにつき 1 クエリ、`take 100` でサンプリング
   - 既存の有用なクエリが `hunts/` にあれば再利用
4. **ヒット時の追加調査**
   - エンティティ(ユーザー/デバイス/プロセス)を抽出
   - 周辺イベント(±10 分)を取得
5. **脅威インテル参照**
   - `ThreatIntelligenceIndicator` に該当があるか確認
   - スキル [skills/threat-intel-enrichment/SKILL.md](../../skills/threat-intel-enrichment/SKILL.md) の手順
6. **保存と出力**
   - 実行クエリを `hunts/<YYYY-MM-DD>-<ioc-slug>.kql` として保存
   - レポートを `reports/hunts/<YYYY-MM-DD>_<ioc-slug>.md` に保存

## アウトプット

- ヒット件数のサマリ表(テーブル × 件数)
- 影響を受けた可能性のあるエンティティ一覧(バックティック付き)
- 追加調査の推奨事項
