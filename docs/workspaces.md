# Sentinel ワークスペース一覧

エージェントが MCP `mcp_microsoft_sen_query_lake` 等を呼ぶ際の `workspaceId` リファレンス。

| 名称 | Workspace ID | 用途 |
| --- | --- | --- |
| `log-sentineldatalake` | `<workspace-id>` | 主データレイク。Sentinel / Defender XDR / Entra / Azure / 一部カスタムログ |

> 単一ワークスペース構成のため、MCP 呼び出し時は `workspaceId` を省略可能(自動既定)。  
> 複数化した場合は本表を更新し、エージェントには明示的に渡すこと。

## 取り込み確認 (Heartbeat / Usage)

- データ取り込み状況: `Usage | summarize sum(Quantity) by DataType, bin(TimeGenerated, 1d)`
- ワークスペース健全性: `SentinelHealth | where TimeGenerated >= ago(1d)`

## 更新ポリシー

- 新規ワークスペース追加時は本ファイルと [skills/sentinel-kql-authoring/workspace-tables.md](../skills/sentinel-kql-authoring/workspace-tables.md) を同時に更新する
- Last verified: 2026-05-04
