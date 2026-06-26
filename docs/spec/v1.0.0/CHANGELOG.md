# Changelog

本仕様は SemVer に準拠します。v1.x 系で破壊的変更を行いません。

## [Unreleased / 1.0.0-draft] — 2026-06-26 (Transport / Trust spec 任意拡張を追加)

### Added
- **3 層構成 (Data / Transport / Trust) を導入**
  - Layer 1 (Data) は今までと同じ JSON Schema 仕様 (Basic 適合)
  - Layer 2 (Transport) で OAuth 2 サーバー間直接通信 (Connected 適合)
  - Layer 3 (Trust) で JWT 署名による真正性証明 (Verified 適合)
- **transport.md** — OAuth 2.0 Authorization Code Flow + PKCE / REST エンドポイント / Capability discovery / レート制限 / RFC 7807 エラーフォーマット
- **trust.md** — JWS (RFC 7515) / JWKS (RFC 7517) / 推奨アルゴリズム ES256 / EdDSA / 鍵ローテーション / 検証フロー
- **conformance.md** — Basic / Connected / Verified の宣言・相互運用マトリクス・取り込み判定フロー
- **examples/well-known.example.json** — `/.well-known/achievement-spec` のサンプル
- **README §3.1 で 3 層構成の関係図を追加**
- **README「責務外」セクションで Transport / Trust spec による補完を明示**

### Rationale
- 「改竄耐性」をコア原則から削除した結果、企業採用の真正性要件に応えられなくなった
- ユーザー介入を排除した OAuth 2 + JWT 署名通信なら本当に機械的真正性が成立する
- 個人開発者の参加障壁を上げないため、Transport / Trust は **任意拡張** とし Basic 適合は Data spec のみで十分
- Verified サイトでも Basic サイトとは JSON ファイル経由で相互運用可能 (フォールバック)

### Notes
- Data / Transport / Trust の 3 spec は独立してバージョニング
- 既存の Basic 実装は v1.x 系で必須要件が増えないことを保証
- Transport / Trust は CC0、誰でも実装可能



### Changed (破壊的、ドラフト段階のため許容)
- **コア原則から「改竄耐性」を削除** (6 原則 → 5 原則)
  - 「sha256 で識別」は照合相手 (公式ハッシュレジストリ等) が未定義で実装不能だった
  - PNG/WebP エンコーダ差・メタデータチャンク差で sha256 はピクセル同一でも変動する
  - 取り込み側サイトに「他人任せ」を連鎖させると誰も実装せず機能不全になるため、降格ではなく **削除** を選択
- **§7.3 を「同一性の判定について」に書き換え** — 「identification は achievement_uid のみ」「サムネは表示専用」と明確化
- **§10.4 を「取り込み時の運営審査」に書き換え** — 機械的真正性判定は本仕様の責務外とし、運営ワークフローのガイドラインに

### Removed
- sha256 バイナリ照合による識別ルール (§7.3 / §10.4 / Quick Start 擬似コードから削除)
- 「公式実績ハッシュテーブル」への参照 (§10.4)

### Notes
- スキーマ自体は無変更 (thumbnail / thumbnails_extra は表示用フィールドとして残る)
- README §12 簡素化方針表の Thumbnail 行も理由を「表示専用方針」に書き換え
- 「機械的真正性判定」を希望する実装サイトは独自に Ed25519 署名・well-known レジストリ等を上位レイヤとして導入してよい



## [Unreleased / 1.0.0-draft] — 2026-06-26 (多言語対応 / Internationalization)

### Added (破壊的、ドラフト段階のため許容: primary_lang を required に)
- **Achievement に `primary_lang` (BCP 47 タグ) を required で追加**
  - 「世界中での利用」を前提とする仕様で title/description の言語が不明だった根本的欠陥を解消
  - 値例: `ja`, `en`, `en-US`, `zh-Hant`, `zh-Hans`, `pt-BR`, `es-419`
- **Achievement に `title_i18n` / `description_i18n` 追加** (任意マップ)
- **Achievement.item_refs に `title_i18n` / `maker_i18n` 追加** (任意マップ)
  - 「ファイナルファンタジーVII」⇔「Final Fantasy VII」を同一作品として扱える
- **UserAchievement.achievement_snapshot に `title_i18n` / `description_i18n` 追加**
  - スナップショットだけで多言語表示可能 (§1 自己完結性の強化)
- **UserAchievement.item_refs に `title_i18n` / `maker_i18n` 追加**
- **UserAchievement に `comment_lang` 追加** (任意、ユーザーコメントの言語を BCP 47 で明示)
- **README §7.5 Internationalization** 全面新設 (構造・ルール・表示側ガイド・識別との関係)

### Rationale
- ActivityPub の `nameMap` パターンに準じた並列マップ設計
- 既存フィールド (`title` 等) は primary_lang での値として後方互換維持
- 識別は引き続き achievement_uid + thumbnail sha256 で行い、テキストは表示用と明確化
- v1.0 凍結後だと意味論の後付けでデータ汚染リスクが高いため、draft 段階で入れた

## [Unreleased / 1.0.0-draft] — 2026-06-26 (thumbnail 複数サイズ許容)

### Added
- **Achievement に `thumbnails_extra` フィールド追加** — 32/64/128/256/512 の追加サムネを任意で持てる
  - 各 entry に `size` + `data_uri` (max 80KB)、配列上限 5 件
  - Steam (256x256) / Xbox (1080) / PSN / RetroAchievements (64) 等の既存サイトからのインポートで強制ダウンサンプリングが不要に
- **README §7 全面改訂** — 16x16 reference + 複数サイズ許容の意図と運用ルールを明文化
  - sha256 識別は 16x16 reference 優先、無ければ最大 size を使う
  - `achievement_snapshot` には extras を含めない (JSON 肥大化防止)
  - `thumbnails_extra` に 16x16 相当を入れることは禁止 (reference は `thumbnail` フィールドに統一)

### Rationale
- 16x16 固定はレトロゲーム文化との親和性は強いが、現代ゲーム (PS5/Xbox 等) からの取り込みで情報量不足
- §1 「アンチ独占」「自己完結性」と整合させるには既存サイトのアイコン規格を受け入れる必要がある
- 16x16 を reference (識別と最低保証表示) として残しつつ extras で拡張する両立設計を採用

## [Unreleased / 1.0.0-draft] — 2026-06-26 (P2 磨き)

### Changed (破壊的、ドラフト段階のため許容)
- **Achievement の `source_site` を required から外し optional 化**
  - `uid` の namespace prefix と冗長 (§12 簡素化方針との整合)
- **Achievement の `created_by` を required から外し optional 化**
  - CC0 は帰属表示を要求しない。クレジットしたい場合は `attribution` フィールドを使う設計に統一
- **両 schema の `item_refs.platform` に `^[A-Z0-9_]{2,16}$` pattern を追加**
  - フリーテキスト (例: "PlayStation", "プレステ") を排除し相互運用性を担保
  - 推奨値リストを README §10.1.1 に追加
- **Achievement の `conditions[].type` の enum 固定を廃止し `^[a-z][a-z0-9_]{1,31}$` の自由 identifier に**
  - boss_defeat / chapter_complete 等の追加を MINOR バージョンを待たず可能に
  - 推奨値リストは description に列挙

### Added
- **README §10.1.1** — platform 推奨値リスト (FC/NES, SFC/SNES 等のリージョン別命名併存ポリシー含む)
- **README §10.1.2** — proof_url 推奨ホワイトリスト (SSRF / 改竄 / 短命ホスト対策)

## [Unreleased / 1.0.0-draft] — 2026-06-26 (P1 採用障壁の解消)

### Changed (破壊的、ドラフト段階のため許容)
- **UserAchievement の `item_refs.local_id` を required から外す**
  - サイト B が作った UserAchievement をサイト C が import したとき、サイト B 固有の internal ID が永続的に残る問題を解消
  - 「エクスポート時に内部運用フィールドを削除」(§10.2) との整合
- **UserAchievement の `created_at` を required に追加**
  - サイト跨ぎで「いつ取り込まれたか」を追跡可能にするため
- **両 schema の `external_ids` を `additionalProperties: true` に変更**
  - Steam / Xbox / PSN 等の vendor 固有 ID を MINOR バージョンを待たずに追加可能に
  - 推奨キー列挙はそのまま維持

### Added
- **README §9.5 Quick Start** — ajv-cli / Python jsonschema での validate コマンド、import / export の最小擬似コード
- **README §10.3 / §11 に前方互換ポリシー明文化** — v1.0.0 実装は v1.x データを受理し、未知フィールドを reject しない

## [Unreleased / 1.0.0-draft] — 2026-06-26 (P0 整合性修正)

### Changed (破壊的、ドラフト段階のため許容)
- **データのデフォルトライセンスを `CC-BY-SA-4.0` → `CC0-1.0` に変更**
  - 「データはユーザーに帰属」「サイト跨ぎで持ち運べる」原則と SA の継承義務が衝突するため
  - 達成事実は著作物ではない (UserAchievement に license を設けない理由と整合)
  - 営利サイトの取り込み障壁を消すことで「アンチ独占」「永続性」を実効化
- **achievement.schema.json の `uid` pattern を `[A-Za-z0-9_:-]+` に拡張** (UserAchievement.achievement_uid と統一)
  - README §5 の例 `gamesoft-navi.com:ach:default:cleared:17602` が valid になるよう修正
- **両 schema の `$id` を中立 URL (`https://ponta0321.github.io/open-achievement-spec/`) に変更**
  - 旧 URL は reference 実装サイト固有 (`gamesoft-navi.com`) で公開仕様の正本 URL として不適切

### Added
- `export.schema.json` — エクスポート JSON エンベロープスキーマ (top-level の `schema_version` / `source_site` / `exported_at` / `user_achievements`)
- `examples/export.example.json` — エクスポート形式のサンプル

## [Unreleased / 1.0.0-draft] — 2026-06-25 (簡素化改訂)

### Changed (破壊的、ドラフト段階のため許容)
- **UserAchievement の自身ID を `uid` → `record_uid` に改名**
  - 裸の `uid` は「何の uid か」が JSON 単体では曖昧だったため
  - `achievement_uid` と対称な命名で混同を防止
- **Thumbnail を `data_uri` のみに簡素化**
  - `kind` / `preset_id` / `palette` / `license` / `attribution` / `alt` を撤去
  - 識別は sha256 バイナリ照合に統一 (改竄耐性確保)

### Removed (簡素化、共通仕様としての無駄を排除)
- UserAchievement の `license` (Achievement に従う)
- UserAchievement の `handle_name` (record_uid の名前空間で発行元判明)
- UserAchievement の `verification` (v1.0.0 では self_report 固定)
- UserAchievement レコード単位の `schema_version` / `source_site` (トップに1つで十分)
- エクスポート JSON トップの `count` (配列長で代替可能)
- Achievement の category 等を UserAchievement に複製していた冗長フィールド全般

### Added (ドラフト初版で既に追加済み)
- `achievement.schema.json` v1.0.0
- `user_achievement.schema.json` v1.0.0
- `examples/` サンプル 2 件
- README / CONTRIBUTING / LICENSE
- 内部実装仕様 `private/spec/moderation.md` `private/spec/import.md` `private/spec/aggregation.md` (公開対象外)

### Notes
- 本ドラフトは MVP 6ヶ月運用後 (2026-12 頃) に **v1.0.0 として凍結** する予定
- 凍結前 (draft 期間) は破壊的変更を行う場合がある
- 凍結後は v1.x 系で破壊的変更を一切行わない
