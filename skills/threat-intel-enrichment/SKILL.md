# SKILL: Threat Intel Enrichment

**いつ使うか**: IOC(IP / Hash / Domain / URL)の素性を確認したいとき。

## 確認の優先順位

1. **テナント内 TI**(`ThreatIntelligenceIndicator`)— 既に取り込まれているか
2. **Microsoft Defender Threat Intelligence**(製品 UI / API)
3. **公開 OSINT**(VirusTotal, AbuseIPDB, Shodan, urlscan.io)
4. **業界 ISAC / 政府機関フィード**(JPCERT/CC, CISA 等)

## 観点

| IOC 種別 | 確認すること |
| --- | --- |
| IP | 所有者(ASN)、地理、過去の悪性履歴、Tor / VPN / Proxy / Cloud かどうか |
| Domain | 登録日、登録者、TLD、DNS 履歴、サブドメイン、homoglyph |
| URL | リダイレクト先、ホスト、SSL 証明書、コンテンツタイプ |
| Hash | ファイル名/種別、初出時刻、同種マルウェアファミリ、署名 |

## テナント内検索の例

```kusto
ThreatIntelligenceIndicator
| where TimeGenerated >= ago(90d)
| where NetworkIP == "192.0.2.10"
   or Url has "evil.example"
   or FileHashValue == "abcd..."
| project TimeGenerated, Description, ConfidenceScore, ThreatType, Active, ExpirationDateTime
```

## OSINT 利用時の注意

- **アップロード型サービス(VirusTotal の File 検索など)に顧客ファイルを直接送らない**
  - ハッシュ検索のみ可、本体送信は事前承認
- 取得した情報は**出典 URL を必ずレポートに記載**
- 商用 API キーが必要な場合は環境変数 / シークレットストアから読む(コード埋め込み禁止)

## エンリッチメントの最小セット

- IP: ASN, Country, IsCloud, IsTor, FirstSeen, MalwareFamilies, RecentReports
- Domain: Registrar, CreationDate, Resolutions(直近30日), Categories
- Hash: KnownAs, FirstSubmission, DetectionRatio, Sandbox 解析サマリ

## 出力例

```markdown
### IOC: `192.0.2.10`
- ASN: AS64500 (Example Hosting)
- Country: NL
- 種別: VPS / Bulletproof Hosting の評判あり
- 過去の悪性履歴: AbuseIPDB 信頼度 95% (出典: https://...)
- テナント内出現: `DeviceNetworkEvents` で過去 30 日 17 件
- 推奨: ファイアウォールで Block 候補(承認要)
```
