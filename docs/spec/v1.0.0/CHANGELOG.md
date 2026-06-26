# Changelog

本仕様は SemVer に準拠します ([README §11](README.md))。v1.x 系で破壊的変更は行いません。
凍結前 (1.0.0-draft 期間) は破壊的変更を行う場合があります。

形式は [Keep a Changelog](https://keepachangelog.com/) に準拠。日付は ISO 8601 (UTC ベース)。

---

## [Unreleased] — 1.0.0-draft

### Added (新機能)

#### 3 層構成 (Data / Transport / Trust) の導入
- Layer 1 (Data) は従来の JSON Schema 仕様 (Basic 適合)
- Layer 2 (Transport) で OAuth 2 サーバー間直接通信 (Connected 適合)
- Layer 3 (Trust) で JWT 署名による真正性証明 (Verified 適合)
- 3 spec は独立バージョニング、Basic 実装は v1.x 系で必須要件が増えない
- `transport.md` / `trust.md` / `conformance.md` / `transport-envelope.schema.json` / `examples/well-known.example.json` を追加
- README §3.1 に 3 層構成の関係図

#### 多言語対応 (Internationalization)
- Achievement に `primary_lang` (BCP 47 タグ) を required で追加
- Achievement / item_refs に `title_i18n` / `description_i18n` / `maker_i18n` 追加
- UserAchievement.achievement_snapshot にも i18n マップ
- UserAchievement に `comment_lang` 追加
- README §7.5 に Internationalization セクション新設 (ActivityPub `nameMap` 流)

#### Thumbnail 複数サイズ許容
- `thumbnails_extra` (32/64/128/256/512、最大 5 件、各 ~55KB)
- Steam (256x256) / Xbox (1080x1080) / RetroAchievements (64x64) からの取り込みでダウンサンプリング不要

#### Quick Start (§9.5)
- ajv-cli / Python jsonschema での validate コマンド例
- import / export の最小擬似コード (i18n / Transport / Trust 反映済み)

#### その他
- `export.schema.json` — ファイル交換用エンベロープスキーマ
- `examples/export.example.json` — エクスポート形式のサンプル
- README §10.1.1 platform 推奨値リスト (FC/NES, SFC/SNES リージョン併存)
- README §10.1.2 proof_url 推奨ホワイトリスト
- README §10.3 / §11 前方互換ポリシー明文化

### Changed (変更、破壊的含む)

#### コア原則 6 → 5 (改竄耐性を削除)
- 「sha256 で識別」は照合相手 (公式ハッシュレジストリ等) が未定義で実装不能だった
- PNG/WebP エンコーダ差・メタデータ差で sha256 はピクセル同一でも変動する
- §7.3 を「同一性の判定について」(achievement_uid のみ) に書き換え
- §10.4 を「取り込み時の運営審査」(運営ワークフロー責務) に書き換え
- 機械的真正性が必要な場合は Trust spec で補完する設計

#### ライセンス
- データのデフォルトを `CC-BY-SA-4.0` → `CC0-1.0` に変更
  - SA の継承義務が「サイト跨ぎ移転」原則と衝突するため

#### Schema 整合化
- achievement.uid / record_uid pattern を `[A-Za-z0-9_:-]+` に統一 (階層 ID 対応)
- 両 schema の `$id` を中立 URL (`ponta0321.github.io/game-achievement-common-spec`) に
- UserAchievement.item_refs.local_id を required から外す (サイト跨ぎ汚染回避)
- UserAchievement.created_at を required に追加 (取り込み追跡可能化)
- 両 schema の `external_ids` を `additionalProperties: true` に (vendor 固有 ID 追加)
- 両 schema の `item_refs.external_ids` の null 許可を統一
- Achievement.source_site / created_by を required から外し optional 化
- item_refs.platform に `^[A-Z0-9_]{2,16}$` pattern (フリーテキスト排除)
- conditions[].type の enum 固定を廃止し `^[a-z][a-z0-9_]{1,31}$` の自由 identifier に
- BCP 47 pattern を改良 (`zh-Hant-TW`, `de-CH-1996` 等が通る)
- `$defs` で BCP 47 / i18nMap を共通定義に括り出し (重複 12 箇所を解消)

#### UserAchievement レコード命名
- 自身 ID を `uid` → `record_uid` に改名 (`achievement_uid` と対称化、混同防止)

#### Thumbnail
- `data_uri` のみに簡素化 (`kind` / `preset_id` / `palette` / `license` / `attribution` / `alt` を撤去)
- サムネは表示専用とし識別には使わない方針 (§7.3)

### Removed (削除)
- UserAchievement の `license` (Achievement に従う)
- UserAchievement の `handle_name` (record_uid の namespace で発行元判明)
- UserAchievement の `verification` (v1.0.0 では self_report 固定)
- UserAchievement レコード単位の `schema_version` / `source_site` (トップに 1 つで十分)
- エクスポート JSON トップの `count` (配列長で代替)
- 「sha256 バイナリ照合による識別ルール」「公式実績ハッシュテーブル」への参照
- 「公開前 / 社外秘期間中」前提の記述全般 (既に GitHub public 公開済み)
- 個人メールアドレス連絡先 (`gamesoft.navi@gmail.com`)

### Notes
- 本ドラフトは MVP 6ヶ月運用後 (2026-12 目標) に **v1.0.0 として凍結** する予定
- 凍結後は v1.x 系で破壊的変更を一切行わない
- 凍結後の MINOR (v1.1.0+) は「任意フィールド追加」「enum 値追加」のみ
- スキーマ URL の `$id` は GitHub Pages 有効化後に解決可能

---

## 凍結後のリリース履歴

(v1.0.0 凍結後にここへ追加します。)
