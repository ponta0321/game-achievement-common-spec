# Contributing

## バグ報告・改善提案

GitHub Issues で受け付けます。内容に応じて 2 種類のテンプレートのいずれかを使ってください。

### 軽い報告用 (typo / 質問 / 軽微な改善)

```
[対象] Data spec / Transport spec / Trust spec / Conformance / その他

[内容]
```

### 仕様変更提案用 (新フィールド / 破壊的変更 / 適合性影響あり)

```
[対象 spec]
- Data / Transport / Trust / Conformance / その他

[種類]
- バグ (仕様矛盾) / 提案 (任意追加) / 破壊的変更提案 (v2.0.0 候補)

[内容]
(具体的に)

[影響範囲]
- 既存実装への影響:
- 既存データへの影響:
- 適合性レベルへの影響 (Basic / Connected / Verified):
```

採用相談・実装 Q&A は [GitHub Discussions](https://github.com/ponta0321/game-achievement-common-spec/discussions) (有効化後) を推奨。

## Pull Request

- 軽微な修正 (typo / 例示の追加 / 文言改善) は直接 PR で
- 仕様変更を伴う場合は Issue で議論後に PR

## バージョニングの方針

- **MAJOR (v2.0.0)** — 破壊的変更。新ディレクトリ `docs/spec/v2.0.0/` で並行発行
- **MINOR (v1.1.0)** — 後方互換な追加 (任意フィールド / enum 値追加)
- **PATCH (v1.0.1)** — 文言修正のみ

**v1.0.0 凍結後、v1.x 系では絶対に破壊的変更しません**。draft 期間 (現在) は破壊的変更を行う場合があります (README §11 参照)。

## 議論の場

- 仕様議論: GitHub Discussions
- 実装相談: 各実装プロジェクトの Issue
