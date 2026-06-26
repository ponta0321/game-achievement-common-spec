# Contributing

## バグ報告・改善提案

GitHub Issues で受け付けます。テンプレート:

```
### 対象 spec
[ ] Data spec (achievement / user_achievement / export)
[ ] Transport spec (OAuth 2 / REST)
[ ] Trust spec (JWS / JWKS)
[ ] Conformance (適合性レベル・相互運用)
[ ] その他 (CHANGELOG / README / examples 等)

### 種類
[ ] バグ (仕様の矛盾・誤記)
[ ] 提案 (任意フィールド追加 / 新エンドポイント等)
[ ] 破壊的変更提案 (v2.0.0 候補)
[ ] 採用相談 / Q&A

### 内容
(具体的に)

### 影響範囲
- 既存実装への影響
- 既存データへの影響
- 適合性レベルへの影響 (Basic / Connected / Verified)
```

## Pull Request

- 軽微な修正 (typo / 例示の追加 / 文言改善) は直接 PR で
- 仕様変更を伴う場合は Issue で議論後に PR

## バージョニングの方針

- **MAJOR (v2.0.0)** — 破壊的変更。新ディレクトリ `docs/spec/v2.0.0/` で並行発行
- **MINOR (v1.1.0)** — 後方互換な追加 (任意フィールド / enum 値追加)
- **PATCH (v1.0.1)** — 文言修正のみ

**v1.x 系では絶対に破壊的変更しません**。

## 議論の場

- 仕様議論: GitHub Discussions
- 実装相談: 各実装プロジェクトの Issue
