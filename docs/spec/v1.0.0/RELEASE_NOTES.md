# Release Notes — v1.0.0

> このファイルは GitHub Release 作成時に貼り付ける説明文の準備です。実際の公開時にユーザーが GitHub Release ページで使用します。

---

## Game Achievement Common Spec v1.0.0

ゲームの達成記録 (アチーブメント) を**特定企業・特定サイトに依存しない形で共有する**オープン標準の v1.0.0 を公開しました。

THE CREW (Ubisoft, 2024 サービス終了) のように運営の判断ひとつで数千時間の積み重ねが消滅する事態を、サイトを跨いで持ち運べるデータ標準により防ぐことを目的としています。

### 主な特徴

#### 3 層構成 (Data / Transport / Trust)
- **Basic 適合 (Data spec)**: JSON ファイル import/export だけで参加可能
- **Connected 適合 (Transport spec)**: OAuth 2 でサーバー間直接通信
- **Verified 適合 (Trust spec)**: JWT 署名で機械的真正性保証

個人開発者は Basic だけで完結、企業は 3 層すべてを利用可能。

#### 多言語対応
- ActivityPub の `nameMap` パターンに準じた並列翻訳マップ
- 全テキストフィールドに `primary_lang` (BCP 47) と `*_i18n` を提供

#### ヘビーユーザー対応
- 単一 JSON 形式 (export.schema.json) と NDJSON 形式 (export-ndjson-meta.schema.json) の両対応
- ストリーミング処理可能、10 万件規模に対応
- API 経路は pagination 標準対応

#### 公式プラットフォーム ID 対応
- Steam / PSN / Xbox / RetroAchievements 等の Achievement ID を `external_ids` で保持
- 機械的同一性判定が可能

### コア原則 (5 つ)

1. **データはユーザーに帰属** — どのサイトに記録しても本人が常にエクスポート/移転できる
2. **アンチ独占** — 単一サイト・単一企業に集約させない設計
3. **永続性** — 1 サイトが消滅しても他サイトでデータが活き続ける
4. **自己完結性** — 1 レコードだけ取り出しても何の作品の何の実績か表示可能
5. **後方互換** — v1.0.0 で凍結、破壊的変更は v2.0.0 として並行発行

### ライセンス

- 仕様書・JSON Schema: **CC0-1.0** (パブリックドメイン相当、商用利用可、フォーク禁止条項なし)
- 生成データのデフォルト: **CC0-1.0** (`license` フィールドで任意の SPDX ID を選択可)

### Getting Started

最小実装 (Basic) は以下のスキーマだけ追えば完結します:

- [achievement.schema.json](docs/spec/v1.0.0/achievement.schema.json)
- [user_achievement.schema.json](docs/spec/v1.0.0/user_achievement.schema.json)
- [export.schema.json](docs/spec/v1.0.0/export.schema.json)

[README §9.5 Quick Start](docs/spec/v1.0.0/README.md#95-quick-start) に validate コマンドと擬似コードがあります。

### 公開後

- reference 実装 ([gamesoft-navi.com](https://gamesoft-navi.com/)) の対応状況は GitHub Discussions で随時公開
- コミュニティからのフィードバックを受けて v1.1.0 / v2.0.0 を将来検討
- v1.x 系では破壊的変更を行わない

### 開発の進め方

- バグ報告・改善提案: [GitHub Issues](https://github.com/ponta0321/game-achievement-common-spec/issues)
- 仕様議論: [GitHub Discussions](https://github.com/ponta0321/game-achievement-common-spec/discussions) (有効化後)
- 凍結後 v1.x で破壊的変更は行いません

---

## English Summary

Game Achievement Common Spec v1.0.0 is published — an open standard (CC0-1.0) for sharing game achievement records across sites without vendor lock-in.

### Three Conformance Levels

- **Basic**: JSON file import/export only (Data spec)
- **Connected**: OAuth 2 server-to-server (Data + Transport spec)
- **Verified**: JWT signature verification (Data + Transport + Trust spec)

### Key Features

- **i18n built-in**: BCP 47 + parallel translation maps (ActivityPub nameMap pattern)
- **Heavy user support**: NDJSON format for streaming, pagination for API
- **Platform achievement IDs**: Steam, PSN, Xbox, RetroAchievements, etc.
- **Site disappearance resilience**: Multi-site distribution + persistent uid identifiers

### Files

- 4 JSON Schemas (Data + Transport)
- 5 example files (Achievement / UserAchievement / Export / NDJSON Export / well-known)
- 3 spec markdown files (Transport / Trust / Conformance)

CC0-1.0 license, no fork restrictions, commercial use OK.
