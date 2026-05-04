# agents/

VSCode 内で定型化したプロセスを、外部実行エージェントへ移管するためのアセット置き場。

## サブディレクトリ

- `copilot-studio/` — Microsoft Copilot Studio エージェント定義(トピック、Knowledge、アクション)
- `security-copilot/` — Microsoft Security Copilot エージェント / Promptbook

## 移管フロー(推奨)

```
[1] VSCode で対話的に検証 (.github/prompts/)
        │
        ▼
[2] runbooks/ に手順をまとめる(人間が安定運用できる粒度に)
        │
        ▼
[3] agents/copilot-studio/ または agents/security-copilot/ に
    エージェント定義として落とし込む
        │
        ▼
[4] ステージング環境でテスト → 本番デプロイ(承認必須)
```

## 留意点

- 外部エージェントに渡す指示は、本リポジトリの `copilot-instructions.md` の安全境界を**そのまま継承**する
- エージェントの権限は**最小権限**で設計(読み取り専用 → 段階的に拡張)
- 監査ログ(エージェントが何をしたか)を必ず残す
