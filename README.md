# Game Achievement Common Spec

レトロゲームを含む全コンシューマゲームの**達成記録 (アチーブメント)** を、特定企業・特定サイトに依存しない形で共有するためのオープン仕様。

THE CREW (Ubisoft, 2024 サービス終了) のように、運営の判断ひとつでプレイヤーの数千時間の積み重ねが消滅する事態を、**サイトを跨いで持ち運べるデータ標準** によって防ぐことを目的とする。

## コア原則

1. **データはユーザーに帰属** — どのサイトに記録しても、本人が常にエクスポート/移転できる
2. **アンチ独占** — 単一サイト・単一企業に集約させない設計
3. **永続性** — 1サイトが消滅しても他サイトでデータが活き続ける
4. **自己完結性** — 1レコードだけ取り出しても、何の作品の何の実績か表示可能
5. **後方互換** — v1.0.0 で凍結。破壊的変更は v2.0.0 として並行発行

> **注**: 「データの真正性 (改竄されていないこと) の保証」は本仕様の責務外。取り込み側サイトが運営ワークフロー (pending → approved 等) で担保する設計とする。

## 仕様

最新版: [docs/spec/v1.0.0/](docs/spec/v1.0.0/)

本仕様は **3 層構成** (Data / Transport / Trust)。個人開発者は Data だけで完結、企業は 3 層すべてで真正性保証を得られる。

### Layer 1: Data (必須)
- [README](docs/spec/v1.0.0/README.md) — 仕様概要・哲学・全体像
- [achievement.schema.json](docs/spec/v1.0.0/achievement.schema.json) — 実績定義スキーマ
- [user_achievement.schema.json](docs/spec/v1.0.0/user_achievement.schema.json) — ユーザー達成記録スキーマ
- [export.schema.json](docs/spec/v1.0.0/export.schema.json) — エクスポート JSON エンベロープスキーマ
- [examples/](docs/spec/v1.0.0/examples/) — サンプルデータ

### Layer 2: Transport (任意・Connected 以上)
- [transport.md](docs/spec/v1.0.0/transport.md) — OAuth 2 サーバー間直接通信仕様

### Layer 3: Trust (任意・Verified)
- [trust.md](docs/spec/v1.0.0/trust.md) — JWT 署名による真正性証明仕様

### 全体
- [conformance.md](docs/spec/v1.0.0/conformance.md) — Basic / Connected / Verified の宣言と相互運用
- [CHANGELOG](docs/spec/v1.0.0/CHANGELOG.md)
- [CONTRIBUTING](docs/spec/v1.0.0/CONTRIBUTING.md)

## ライセンス

| 対象 | ライセンス |
|---|---|
| 本仕様書・JSON Schema | **CC0-1.0** (パブリックドメイン相当) |
| 仕様に基づき生成されたデータ | **CC0-1.0 デフォルト** (Achievement 単位で `license` フィールドにより任意の SPDX ID を選択可) |

データを CC0 デフォルトにする理由 — 「データはユーザーに帰属」「サイト跨ぎで持ち運べる」原則と、CC-BY-SA の帰属表示・継承義務は摩擦する。実績の達成事実そのものは著作物ではなく、Achievement 定義の短文・16×16 サムネに著作物性が認められるかも不明瞭。CC0 にすることで取り込み側サイトの法的負担を消し、§1「アンチ独占」「永続性」を実効化する。作者が明示的にライセンスを付けたい場合は `license` フィールドで個別指定可能。

## Reference 実装

- [gamesoft-navi.com](https://gamesoft-navi.com/) — 本仕様に準拠したアチーブメント機能を提供
