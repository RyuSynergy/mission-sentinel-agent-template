---
mode: agent
description: MDVM 等の脆弱性アラートを資産価値・露出・悪用可能性で評価
---

# 脆弱性アラートのリスク評価

Microsoft Defender Vulnerability Management(MDVM)等から上がった脆弱性について、組織内コンテキストでリスクを評価します。

## 入力

- **CVE ID または推奨事項 ID**: ${input:cve:例 CVE-2025-12345}

## 評価軸

1. **脆弱性の固有リスク**
   - CVSS v3.1 Score / Severity
   - Exploitability(PoC 公開 / Active Exploitation)
   - CISA KEV カタログ該当
   - Microsoft Exploitability Index
   - 出典: Microsoft Learn / NVD / CISA KEV
2. **組織内の露出**
   - 該当ソフトウェアを持つデバイス数(`DeviceTvmSoftwareInventory`)
   - インターネット露出デバイス数(`DeviceInfo` の Public IP / NetworkAdapters)
   - 特権ユーザー / クラウン環境配下のデバイス数
3. **保護状況**
   - パッチ適用率
   - Workaround 適用状況
   - 関連検知ルールが有効か(`detections/` を grep)
4. **検知 / 痕跡**
   - 同 CVE を悪用する既知 IOC が `ThreatIntelligenceIndicator` にあるか
   - 関連プロセス名 / コマンドラインで `DeviceProcessEvents` を直近 30 日検索

## アウトプット

- `reports/vuln/<CVE-ID>_YYYY-MM-DD.md` に保存
- 総合リスクレーティング(Critical / High / Medium / Low)と根拠
- 推奨アクション:
  - 緊急パッチ対象デバイス一覧
  - Workaround / 補償的検知ルール案
  - **本番ルール作成・パッチ展開は人間承認必須**
