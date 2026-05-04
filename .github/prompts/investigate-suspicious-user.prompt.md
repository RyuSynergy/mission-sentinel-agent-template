---
mode: agent
description: あいまいな「怪しいユーザーを調べて」依頼に対する、入口別分析+相関ワークフロー
---

# 怪しいユーザー調査(あいまい依頼用)

ユーザーから「怪しいユーザーを調べて」「最近不審なアカウントは?」のような**広範な依頼**を受けたときに使用する。

## 守るべきこと

- 必ず [skills/suspicious-user-investigation/SKILL.md](../../skills/suspicious-user-investigation/SKILL.md) を**最初に読み込む**
- 実在テーブルかどうかを [skills/sentinel-kql-authoring/workspace-tables.md](../../skills/sentinel-kql-authoring/workspace-tables.md) で確認
- 全クエリに**時間範囲を明示**(既定 30 日)
- インシデント番号は **XDR `ProviderIncidentId` を主、Sentinel `IncidentNumber` を補助**で記載

## 入力

- **ユーザー名(任意)**: ${input:user:UPN または local-part(空欄なら全社で候補抽出)}
- **期間**: ${input:lookback:既定 30d}
- **深掘り対象数**: ${input:topN:候補の上位何件を深掘りするか(既定 1)}

## 実行ステップ

1. **Discover フェーズ**(ユーザー未指定時のみ)
   - SKILL §1 の入口別候補抽出クエリ(Entra / VPN / OnPrem-AD / Endpoint / XDR-Alerts / Email)を**並列実行**
   - SKILL §1.2 の統合クエリで Top N を提示
   - ユーザーへ深掘り対象を確認(既定で上位 1 件)
2. **Per-Surface Drilldown**(対象ユーザーごとに繰り返し)
   - SKILL §3 の入口別クエリを**入口ごとに独立実行**
   - 各入口で「期間 / 件数 / 異常指標 / 関連アラート / 暫定評価」を表にまとめる
3. **Cross-Surface Correlation**
   - SKILL §4.1 の `union` で**統合タイムライン**を作成
   - SKILL §4.2 の橋渡しシグナル(VPN→AD、ブルート→成功、デバイス→外部認証)を必ずチェック
   - SKILL §4.3 に従い**Mermaid で入口関連図**を描く
4. **最終レポート出力**
   - 保存先: `reports/incidents/YYYY-MM-DD_xdr-<id>_<user-slug>.md`
   - 構造: SKILL §5 のフォーマット
   - インシデント番号表記: `XDR #<id> (Sentinel #<num>)`

## アウトプット

- レポートのファイルパス
- 候補ユーザー Top N(表)
- 最重要対象の評価(TP/BP/FP/Inconclusive + 確度)
- 推奨アクション(即時 / 中期 / 検知改善)
