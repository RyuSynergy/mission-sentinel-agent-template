# SKILL: MITRE ATT&CK Mapping

**いつ使うか**: アラート / インシデント / 検知ルールに ATT&CK Tactic / Technique を付与するとき。

## 基本原則

1. **観測された行為**だけをマッピングする(推測でマッピングしない)
2. **Technique と Sub-technique を区別**する(`T1059` ではなく `T1059.001 PowerShell` まで特定)
3. 1 つのアラートに **複数 Technique** が当たるのが普通(無理に 1 つに絞らない)
4. **Tactic は Technique の親**として併記する(例: Execution / T1059.001)

## マッピング手順

1. **観測事実を箇条書きで整理**
   - 例: `powershell.exe -enc <base64>` が実行された / 親プロセスが `winword.exe`
2. **行為の意図(Tactic)を判断**
   - 実行された → Execution
   - 永続化のためにレジストリが書かれた → Persistence
3. **Technique 候補を選定**
   - ATT&CK Navigator / 公式マトリクスで該当を探す
   - `microsoft_docs_search` で「Defender alert <Tactic>」のように補助検索
4. **証拠との対応を明記**
   - 「`T1059.001 PowerShell`: `powershell.exe -enc` の実行を `DeviceProcessEvents` で観測(`TimeGenerated=...`)」

## よく使う Technique 早見表

| シナリオ | Technique |
| --- | --- |
| 不審な PowerShell | `T1059.001 PowerShell` |
| Base64/難読化 | `T1027 Obfuscated Files or Information` |
| 永続化(Run キー) | `T1547.001 Registry Run Keys / Startup Folder` |
| RDP 横展開 | `T1021.001 Remote Desktop Protocol` |
| SMB 横展開 | `T1021.002 SMB/Windows Admin Shares` |
| パスワードスプレー | `T1110.003 Password Spraying` |
| トークン窃取 | `T1528 Steal Application Access Token` |
| OAuth 同意フィッシング | `T1528` + `T1566 Phishing` |
| LOLBin | 該当 Technique + `T1218 System Binary Proxy Execution` |
| C2 ビーコン | `T1071.001 Web Protocols` |
| データ持ち出し(クラウド) | `T1567.002 Exfiltration to Cloud Storage` |

## 出力フォーマット

レポートには次の形で記載:

```markdown
## MITRE ATT&CK マッピング
- `T1059.001 PowerShell` (Execution) — `powershell.exe -enc` 実行
- `T1027 Obfuscated Files or Information` (Defense Evasion) — Base64 でコマンドラインを難読化
```

## 注意

- ATT&CK バージョンは年 1〜2 回更新される。古い ID を使い続けないよう、参照時は最新版を確認
- **`Other` を多用しない**(マッピングを諦める前にもう一度公式マトリクスを当たる)
