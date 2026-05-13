# Sentinel 分析インストラクション

このワークスペースでは Microsoft Sentinel ワークスペース `xdrworkshoplaws`（ID: `49afd369-5dfc-4739-9309-58a19fb75786`）を使用する。

## 必須参照テーブル

Sentinel のログ分析を行う際は、以下のカスタムテーブルを**必ず**分析対象に含めること。

| テーブル名 | 用途 |
|---|---|
| `VPNAuthEvents_CL` | Cisco ASA VPN の認証ログ（成功・失敗・ブルートフォース検出） |
| `GatewayTraffic_CL` | ゲートウェイ経由のネットワークトラフィックログ |

## クエリ時の注意事項

- `VPNAuthEvents_CL` の主要フィールド: `SourceUserName`, `SourceIP`, `EventOutcome`, `DeviceAction`, `Message`
- `GatewayTraffic_CL` の主要フィールド: `SourceIP`, `DestinationIP`, `Action`, `UserName`
- 怪しいユーザーを探す際は、上記 2 テーブルと `SigninLogs`・`SecurityAlert` を必ず横断して確認すること
