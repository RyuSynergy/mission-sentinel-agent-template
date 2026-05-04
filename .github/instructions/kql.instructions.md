---
applyTo: "**/*.kql"
---

# KQL 作成規約

このリポジトリで `.kql` ファイルを生成・編集するときの規約です。

## 1. ファイルヘッダ

すべての `.kql` ファイルの先頭に次のコメントブロックを置く。

```kusto
// Title       : <短いタイトル>
// Purpose     : <検知/ハンティング目的を1〜2文>
// DataSource  : <SecurityEvent / SigninLogs / DeviceProcessEvents など>
// MITRE       : <Txxxx Technique Name>(該当する場合)
// Author      : <作成者>
// LastUpdated : <YYYY-MM-DD>
// Notes       : <既知の False Positive / チューニング履歴>
```

## 2. クエリ構造

1. 時間範囲は **`let lookback = ago(7d);`** のように先頭で `let` 定義する
2. パラメータ化できる値(IP、ユーザー、しきい値)はすべて先頭の `let` で定義
3. `project` で**必要なカラムだけに絞る**(`*` は禁止)
4. `summarize` の前に `where` で絞り込む
5. 時刻カラムは `TimeGenerated` を基本とし、必要なら `bin(TimeGenerated, 1h)` で集計

## 3. パフォーマンス

- `search *` / `union *` は使わない(明示的にテーブルを指定)
- `join` は **小さい側を左** に置く
- `has` / `has_any` を `contains` より優先(インデックス利用)
- 文字列比較は **大文字小文字を区別する `==`** を優先(`=~` は最後の手段)

## 4. 命名規則

- 変数: `camelCase`(例: `suspiciousIps`)
- テーブルエイリアス: 短縮形を避け、テーブル名を素直に
- ブール値カラム: `is` / `has` プレフィックス

## 5. 出力カラムの順序(レポート用)

`TimeGenerated, Severity, Entity(User/Device/IP), Action, Details, SourceTable` の順で `project` する。

## 6. 禁止事項

- 認証情報・トークンをクエリ内にハードコードしない
- `print` で機密情報を出さない
- 取得行が**極端に多くなる**クエリ(数百万行)を `take` なしで実行しない

## 7. 雛形

```kusto
// Title       : Suspicious sign-ins from rare countries
// Purpose     : 通常使われない国からのサインインを検出
// DataSource  : SigninLogs
// MITRE       : T1078 Valid Accounts
// Author      : <name>
// LastUpdated : 2026-05-04
// Notes       : 出張時の正常サインインを除外する必要あり

let lookback = ago(7d);
let rareCountries = dynamic(["KP","IR","SY"]);
SigninLogs
| where TimeGenerated >= lookback
| where ResultType == 0
| where LocationDetails.countryOrRegion in (rareCountries)
| project TimeGenerated, UserPrincipalName, IPAddress, Location=LocationDetails.countryOrRegion, AppDisplayName
| order by TimeGenerated desc
```
