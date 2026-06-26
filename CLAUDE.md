# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ コアルール (毎セッション最初に確認)

1. **言語**: ユーザーとの対話・コメント・コミットメッセージはすべて **日本語**。
2. **git 運用**: `git commit` までは Claude が自動実行する。`git push origin main` は **ユーザーが手動実行** する (Claude Code の安全装置で main への直接 push がブロックされるため)。commit 完了後は「→ `git push origin main` をお願いします」と必ず通知する。
   - **コミットメッセージに `Co-Authored-By: Claude ...` trailer を付けない**。GitHub Contributors に AI 名義が並ぶのを避けるため。共著表記なしで通常通りコミットする。
3. **GitHub アクセス**: `gh` CLI は `ponta0321` アカウントで認証済み (scopes: `gist`, `read:org`, `repo`, `workflow`)。issue/PR/リポジトリ情報の read 系操作は Claude から実行可。

## このリポジトリの性質

**コードを含まないドキュメント/スキーマ専用リポジトリ** です。ビルド・テスト・lint ツールチェーンは存在しません。成果物は `docs/spec/` 配下の Markdown と JSON Schema のみ。

フォーク元: `E:\w\pro\24121701_gamesoft-navi\source` (ゲームソフトナビ本体)。本リポジトリは、そこから「ゲーム実績 (アチーブメント) 共通仕様」部分だけを切り出し、GitHub で公開するためのもの。

## 仕様の中核設計 (編集時に踏まえるべきこと)

仕様の全体像は [docs/spec/v1.0.0/README.md](docs/spec/v1.0.0/README.md) にある。複数ファイルにまたがる重要な不変条件:

- **2 エンティティ構成**: `Achievement` (定義) と `UserAchievement` (達成記録)。スキーマは [achievement.schema.json](docs/spec/v1.0.0/achievement.schema.json) / [user_achievement.schema.json](docs/spec/v1.0.0/user_achievement.schema.json)、サンプルは [examples/](docs/spec/v1.0.0/examples/)。
- **UID 形式**: `<site_namespace>:<kind>:<local_id>` (`kind` は `ach` または `uach`)。自己 ID と参照先 ID を区別するため、フィールド名は **`record_uid` / `achievement_uid` を使い、裸の `uid` は使わない**。
- **自己完結性**: `UserAchievement` には `achievement_snapshot` (title / description / thumbnail.data_uri) を埋め込み、元 Achievement が取得できなくても表示可能にする。
- **Thumbnail は 16×16 ピクセル統一**で `data_uri` のみ保持。識別は **base64 デコード後バイナリの sha256 のみ** で行い、`preset_id` 等の派生情報は持たせない (改竄耐性のため)。
- **ライセンスは Achievement にのみ付与** (`CC-BY-SA-4.0` がデフォルト)。`UserAchievement` には `license` を設けない (個々の達成は知的創作物ではない)。
- **v1.x は凍結**: v1 系では絶対に破壊的変更を入れない。破壊的変更は `docs/spec/v2.0.0/` を並行発行する形で行う。MINOR は任意フィールド/enum 追加のみ、PATCH は文言修正のみ ([CHANGELOG.md](docs/spec/v1.0.0/CHANGELOG.md) / [CONTRIBUTING.md](docs/spec/v1.0.0/CONTRIBUTING.md))。
- **公開版で削除済みのフィールド**を不用意に復活させない。README §12-13 に撤去理由と「内部 DB では持つが公開仕様からは除外する」項目 (`owner_code_hash` / `report_count` / `moderation_status` / `local_db_id` 等) が列挙されている。

## ライセンス

- 仕様書・JSON Schema 本体: **CC0-1.0**
- 仕様に基づき生成されたデータ: **CC-BY-SA-4.0** がデフォルト (Achievement 単位で `license` フィールド指定可)

## GitHub リモート

- **origin**: `https://github.com/ponta0321/game-achievement-common-spec.git` (public)
- 初回 push (`git push -u origin main`) はユーザー手動で実行する必要がある。以降の push も同様に手動。
- Issue / PR の受付は README §16 の通り公開判断 (2026-12 GO/NO-GO) 後を想定。

## 公開状況

README §16 のロードマップ通り、現時点 (2026-06) は **公開判断前 (社外秘)** の段階。Issue/PR 受付や GitHub Discussions は公開後 (2026-12 GO/NO-GO 判断以降) に開始予定。
