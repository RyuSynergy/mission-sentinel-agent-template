---
mode: agent
description: 直近1週間のセキュリティ運用サマリレポートを生成
---

# 週次サマリレポート生成

過去 7 日間の SOC 活動・検知傾向・推奨アクションをまとめたレポートを作成します。

## 入力

- **対象週**: ${input:week:ISO 週番号 例 2026-W18(空欄で直近の週)}

## 実行手順

1. **インシデント統計**
   - `SecurityIncident` から過去 7 日の作成件数を Severity / Status / Classification 別に集計
   - 前週比(増減%)も算出
2. **トップ検知ルール**
   - `SecurityAlert` から発火件数上位 10 ルール
   - False Positive 率(Classification ベース)も併記
3. **トップエンティティ**
   - 関与回数の多いユーザー / デバイス / IP の上位 10
   - **PII はマスク**(`User-001` 形式)
4. **新規 / 注目すべき脅威**
   - 過去 7 日に初出現した IOC(`ThreatIntelligenceIndicator`)
   - 高 Severity の TP インシデント上位 5 件の概要
5. **データ品質**
   - 主要テーブルの取り込み遅延 / 欠損(`Heartbeat`, `Usage`)
6. **未対応事項**
   - Open のまま 72h 超過のインシデント一覧
7. **推奨アクション**
   - チューニング候補ルール(FP 率高)
   - 新規検知の提案(注目イベントから)

## アウトプット

- `reports/weekly/YYYY-WW.md` に保存
- フォーマットは [.github/instructions/incident-report.instructions.md](../instructions/incident-report.instructions.md) の構造を週次用に拡張

## 守ること

- すべての数値に**根拠クエリ**を併記
- グラフ生成が必要なら Mermaid を使う
- 顧客 PII は ID 化して掲載
