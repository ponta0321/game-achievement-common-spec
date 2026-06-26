# Game Achievement Common Spec — v1.0.0

> **Status**: 公開仕様 (CC0-1.0)。Reference 実装: [gamesoft-navi.com](https://gamesoft-navi.com/)

---

## 1. 目的・哲学

レトロゲームを含む全てのコンシューマゲームについて、**プレイヤーの達成記録 (アチーブメント)** を **特定企業・特定サイトに人質に取られない形** で共有可能にする。

THE CREW (Ubisoft, 2024 サービス終了) のように、運営の判断ひとつでプレイヤーの数千時間の積み重ねが消滅する事態を、**サイトを跨いで持ち運べるデータ標準** によって防ぐ。

### コア原則

1. **データはユーザーに帰属** — どのサイトに記録しても、ユーザー本人が常にエクスポート/移転できる
2. **アンチ独占** — 単一サイト・単一企業に集約させない設計
3. **永続性** — 1サイトが消滅しても他サイトでデータが活き続ける
4. **自己完結性** — 1レコードだけ取り出しても、何の作品の何の実績か表示可能 (`achievement_snapshot`)
5. **改竄耐性** — ユーザーがローカルで JSON を編集できる前提で、画像バイナリ sha256 で識別
6. **後方互換** — v1.0.0 で凍結。破壊的変更は v2.0.0 として並行発行

---

## 2. ライセンス

| 対象 | ライセンス |
|---|---|
| 本仕様書・JSON Schema (このリポジトリ) | **CC0-1.0** (パブリックドメイン相当) |
| 仕様に基づき生成されたデータ (achievement / user_achievement) | **CC0-1.0 デフォルト** (Achievement 単位で `license` フィールドにより任意の SPDX ID を選択可) |
| 仕様を実装するコード | 各実装者が自由に選択 (Apache-2.0 / MIT / GPL 等) |

CC0 により、本仕様は **誰でも・どのサイトでも・改変なしに採用可能**。フォーク禁止条項なし。商用利用可。

ライセンスは **Achievement (定義)** にのみ付与する。UserAchievement (達成記録) には license 項目を設けない (個々の達成行為は知的創作物ではないため)。

### 2.1 データを CC0 デフォルトにする理由

初期ドラフトでは生成データを `CC-BY-SA-4.0` デフォルトとしていたが、§1 のコア原則と衝突するため CC0 に変更した:

- **「データはユーザーに帰属」「サイト跨ぎで持ち運べる」と CC-BY-SA の継承義務が衝突** — 取り込み側サイトが派生物全体を CC-BY-SA で公開する義務を負う可能性が出て、営利サイトの取り込み障壁になる
- **達成事実は著作物ではない** — UserAchievement に license を設けない §4.2 の理由と整合
- **Achievement 定義の著作物性も不明瞭** — 80 文字 title / 600 文字 description / 16×16 サムネに著作権が認められるかは法域依存
- **採用障壁の最小化** — JSON Schema / OpenAPI / ActivityPub 等の成功した標準と同じ戦略

作者が明示的にライセンスを付けたい場合 (詳細な description や独自サムネ等) は `license` フィールドで任意の SPDX ID を指定可能。

---

## 3. ファイル構成

```
docs/spec/v1.0.0/
├── README.md                          ← 本ファイル
├── CHANGELOG.md                       ← バージョン履歴
├── LICENSE                            ← CC0-1.0 全文
├── CONTRIBUTING.md                    ← 仕様改訂の提案手順
├── achievement.schema.json            ← 達成項目スキーマ
├── user_achievement.schema.json       ← ユーザー達成記録スキーマ
├── export.schema.json                 ← エクスポート JSON エンベロープスキーマ
└── examples/
    ├── achievement.example.json
    ├── user_achievement.example.json
    └── export.example.json
```

---

## 4. データモデル概要

本仕様は **2 種類のエンティティ**から成る。各フィールドの厳密な型・制約は JSON Schema (`achievement.schema.json` / `user_achievement.schema.json`) を一次情報とし、本節は読みやすさのための要約である。

### 4.1 Achievement (達成項目の定義)

「ある作品に対して定義された目標 (例: 全マテリア収集)」を表す。誰が作成してもよい。複数サイトで独立に同等の Achievement が定義されてもよい (`uid` で識別)。

**必須フィールド:**
- `schema_version` — `"1.0.0"` 固定
- `uid` — グローバル一意 ID (`<site_namespace>:ach:<local_id>`)
- `primary_lang` — テキストフィールドの主言語 (BCP 47, 例: `ja`, `en-US`)。§7.5 参照
- `item_refs` — 対象作品参照 (title + platform 必須)
- `title` — 実績タイトル (primary_lang での値、2〜80 文字)
- `created_at` — 作成日時 (date-time)
- `license` — SPDX ID、デフォルト `CC0-1.0`

**任意フィールド (主要):**
- `source_site` — 発行元サイト URL (uid prefix と冗長なので省略可)
- `title_i18n` / `description_i18n` — `{ "<bcp47>": "..." }` の翻訳マップ
- `description` (≤600 字) / `category` / `difficulty` (1-5) / `tags` (≤10) / `conditions`
- `thumbnail` — **16×16 reference サムネ** (data_uri、≤2KB)
- `thumbnails_extra` — 32/64/128/256/512 の追加サムネ (任意、最大 5 件)。§7 参照
- `created_by` (ハンドル名) / `updated_at` / `attribution`

### 4.2 UserAchievement (ユーザー達成記録)

「あるユーザーが ある Achievement を達成した記録」を表す。**他サイトに持ち込んでも自己完結する**よう、表示に必要な最小情報 (`achievement_snapshot`) を i18n マップ込みで埋め込む。

**必須フィールド:**
- `record_uid` — この記録自身のグローバル一意 ID (`<site_namespace>:uach:<local_id>`)
- `achievement_uid` — 対象 Achievement の uid
- `item_refs` — 達成時の作品参照 (title + platform 必須)
- `achieved_at` — ユーザー申告の達成日 (date)
- `created_at` — DB 登録時刻 (date-time)

**任意フィールド (主要):**
- `achievement_snapshot` — 表示用スナップショット (title / title_i18n / description / description_i18n / thumbnail)
- `comment` (≤400 字) / `comment_lang` (BCP 47) / `proof_url`

UserAchievement には **license フィールドを設けない**。個々の達成行為は知的創作物ではなく、ライセンスは Achievement (定義) に従う。

---

## 5. UID 設計

```
<site_namespace>:<kind>:<local_id>

site_namespace : 発行元サイトのドメイン (lowercase, ASCII)。例: example.com
kind           : "ach"  → Achievement
                 "uach" → UserAchievement
local_id       : 発行元サイト内での一意 ID (英数 + _ - : のみ)
```

例:
- Achievement (UGC): `example.com:ach:1234`
- Achievement (default): `example.com:ach:default:cleared:17602`
- UserAchievement: `example.com:uach:88012`

これにより、複数サイトのデータをマージしても衝突しない。

### 5.1 命名規則 (本仕様内のフィールド名)

UserAchievement レコードは「自身の uid」と「参照先の uid」を両方持つため、混同を避けるため:
- 自身のID: `record_uid`
- 参照先のID: `achievement_uid`

の対称な命名を採用する。**裸の `uid` という名前は使わない**。

---

## 6. Achievement Snapshot — 自己完結性の中核

UserAchievement は、他サイトに取り込まれたときに **元の Achievement レコードが取得できなくても表示可能** でなければならない。そのため、達成時点の表示情報スナップショットを **多言語対応込みで** 埋め込む。

```json
"achievement_snapshot": {
  "title": "ナイツオブザラウンド入手",
  "title_i18n": {
    "en": "Got Knights of the Round"
  },
  "description": "ヒュージマテリアを全て集めて...",
  "description_i18n": {
    "en": "Collect all huge materia and..."
  },
  "thumbnail": {
    "data_uri": "data:image/png;base64,..."
  }
}
```

- `title` 必須 (snapshot 元の Achievement の primary_lang での値)
- `title_i18n` / `description_i18n` 任意。snapshot だけで取り込み先サイトのユーザー言語で表示できるようにするための翻訳マップ
- `description` 任意
- `thumbnail.data_uri` 任意。あれば 16×16 PNG/WebP/GIF の base64 data URI
- `thumbnails_extra` は **snapshot には含めない** (JSON 肥大化防止、元 Achievement を辿る)

---

## 7. Thumbnail 設計

サムネは **16×16 ピクセルを reference (推奨基準)** とし、追加で 32/64/128/256/512 のいずれかの **複数サイズを任意で許容** する。表示サイドは利用可能な最大サイズを選んで描画する。16×16 のみ存在する場合は CSS `image-rendering: pixelated` で 2x/4x/8x に拡大しドット感を保持。

### 7.1 設計の意図

- **コアアイデンティティ**: 16×16 はレトロゲーム文化との親和性、ドット絵の手描きしやすさ、JSON サイズ予算の安定性を担保する reference サイズ
- **アンチ独占の実効化**: Steam (256×256) / Xbox (1080×1080) / PSN / RetroAchievements (64×64) 等の既存サイトからアイコンを取り込む際に、**強制ダウンサンプリング不要**。元解像度を `thumbnails_extra` で保持できる
- **自己完結性は 16×16 で担保**: UserAchievement の `achievement_snapshot.thumbnail` は **常に 16×16 reference のみ**。`thumbnails_extra` をスナップショットに含めない (JSON 肥大化防止、元 Achievement を辿ればよい)

### 7.2 構造

```json
"thumbnail": {
  "data_uri": "data:image/png;base64,..."          // 16x16 reference (必須レベル推奨)
},
"thumbnails_extra": [                                // 任意・最大 5 件
  { "size": 64,  "data_uri": "data:image/png;base64,..." },
  { "size": 256, "data_uri": "data:image/png;base64,..." }
]
```

- `data_uri` のみ。`preset_id` / `kind` / `license` 等の派生情報は持たない
- `thumbnail.data_uri` の `maxLength` は 2000 文字 (≒ 1.4KB バイナリ)
- `thumbnails_extra[].data_uri` の `maxLength` は 80000 文字 (≒ 55KB バイナリ)。合計ペイロードは ~200KB 以下に収めることを推奨
- `size` は `[32, 64, 128, 256, 512]` のいずれか (正方形のみ)
- 同一 size のエントリは複数持たない (実装側は最初のものを採用)

### 7.3 識別の原則 (重要)

実装サイトが「同じ実績のサムネかどうか」を判定する際は、**sha256 バイナリ照合** を使用する。判定の優先順位:

1. **`thumbnail` (16×16 reference) の sha256 を最優先で照合** — 全実装が持つ「べき」reference サイズ。最も信頼できる識別子
2. `thumbnail` が両側に無い (どちらかが省略) → `thumbnails_extra` のうち **両者が共通して持つ size のエントリの sha256** を順に試す (大きい size から)。共通 size が無ければマッチ不能
3. `preset_id` 等の付帯情報による識別は禁止 (改竄可能なため)

`thumbnails_extra` 同士は **size が違えば sha256 も異なる** (リサイズしただけで別ハッシュ)。マッチ不能を避けるため、§7.4 の「自動ダウンサンプリングで 16×16 reference を生成する」を強く推奨する。

### 7.4 16×16 reference を省略してよいか

「Achievement に大きいアイコンしか無い (Steam インポート等) ため 16×16 を作れない」ケースがある。

- **推奨**: 実装サイトが取り込み時に **自動ダウンサンプリング** で 16×16 を生成し `thumbnail` に格納する
- **許容**: `thumbnail` を null とし `thumbnails_extra` のみ持つ。ただし他サイトとの sha256 識別精度は落ちる
- **禁止**: `thumbnails_extra` の中に「16」や「16×16 相当」を入れること (reference は `thumbnail` フィールドに統一)

---

## 7.5 Internationalization (多言語対応)

本仕様は世界中での利用を想定するため、テキストフィールド (title / description / item_refs.title / item_refs.maker) に **言語識別子と並列翻訳マップ** を持つ。設計は ActivityPub の `nameMap` パターンに準じる。

### 7.5.1 構造

```json
{
  "primary_lang": "ja",
  "title": "ナイツオブザラウンド入手",
  "title_i18n": {
    "en": "Got Knights of the Round",
    "zh-Hant": "獲得圓桌騎士"
  },
  "description": "ヒュージマテリアを全て集めて...",
  "description_i18n": {
    "en": "Collect all huge materia and..."
  },
  "item_refs": {
    "title": "ファイナルファンタジーVII",
    "title_i18n": { "en": "Final Fantasy VII" },
    "maker": "スクウェア",
    "maker_i18n": { "en": "Square" }
  }
}
```

### 7.5.2 ルール

- **`primary_lang` は必須** (BCP 47 タグ)。`title` / `description` 等のフリーテキストは `primary_lang` で書く
- **`*_i18n` マップは任意・追加翻訳のみ**。`primary_lang` の値を i18n マップに重複させない (SHOULD NOT、DRY 原則)。重複していた場合は実装は `primary_lang` 側の値を優先する (i18n マップ側を無視) こと
- キーは BCP 47 タグ (`ja`, `en`, `en-US`, `zh-Hant`, `zh-Hant-TW`, `zh-Hans`, `pt-BR`, `es-419`, `de-CH-1996` 等)
- 値が無い言語はキーごと省略する (null 不可)
- attribution / created_by / comment はユーザー記入物なので翻訳しない (`comment_lang` で言語明示のみ)

### 7.5.3 表示側 (取り込みサイト) のルール

```pseudo
function display_title(achievement, user_lang):
    # 1. ユーザー言語と完全一致
    if achievement.title_i18n[user_lang] exists:
        return (achievement.title_i18n[user_lang], user_lang)

    # 2. 言語サブタグだけ一致 (en-US ユーザーに en を表示)
    base = user_lang.split("-")[0]
    for tag, value in achievement.title_i18n:
        if tag.startswith(base):
            return (value, tag)

    # 3. primary_lang にフォールバック
    return (achievement.title, achievement.primary_lang)
```

表示時は HTML で `<span lang="...">` を必ず付与し、スクリーンリーダー・自動翻訳が言語を認識できるようにする。

### 7.5.4 識別との関係

テキストは識別子ではなく **表示用**。同一性判定は引き続き `achievement_uid` + thumbnail sha256 (§7.3) で行う。同じ `achievement_uid` を持つレコードが言語違いの title を持っていても、それは「翻訳の追加」として扱う (実績の同一性は変わらない)。

### 7.5.5 v1.0 で意図的に入れないもの

- **`alternates` 配列 / `locale_variants` 等の重厚な国際化** — 達成条件 (conditions) は言語非依存なので、locale 別レコードを増やす必要はない
- **逆引きインデックス** — 「英語ユーザーが英語タイトルで検索したい」は実装側の検索インフラの責務であり、データ仕様の責務ではない

## 8. 相互運用フロー (想定ユースケース)

### 8.1 ユーザーが自分のデータをエクスポート

```
[サイト A] POST /user_export.php (handle + password)
  → JSON ダウンロード
  → user_achievement の配列 (本仕様準拠)
  → ユーザーがローカル保存
```

### 8.2 別サイトへ持ち込み

```
[サイト B] POST /import (ファイルアップロード + handle + password)
  → schema_version 検証 (1.x 受理、未知フィールドは無視)
  → record_uid 重複検知 (既取込みはスキップ)
  → achievement_uid + thumbnail.data_uri sha256 で識別 (§7.3)
  → 自サイト発行物 / 他サイト発行物 を判別
  → item_refs.external_ids → title+platform → title_i18n.*+platform
     の順で サイト B 側の item に紐付け
  → primary_lang とサイト B のユーザー言語が異なる場合は
     achievement_snapshot.title_i18n / item_refs.title_i18n から
     ユーザー言語の表記を取得 (無ければ primary_lang のまま)
  → 取込み (pending → 運営審査 → approved)
```

### 8.3 サイト消滅時の救済

```
[サイト A] 終了告知 (例: 6ヶ月前)
  → 全ユーザーに export 推奨
  → ユーザーは複数の代替サイトへ import
  → record_uid + handle で重複検知可能
```

---

## 9. エクスポート JSON の構造

トップレベル:

```json
{
  "schema_version": "1.0.0",
  "source_site": "https://example.com/",
  "exported_at": "2026-06-25T11:48:26+09:00",
  "user_achievements": [ /* UserAchievement[] */ ]
}
```

| フィールド | 必須 | 説明 |
|---|---|---|
| `schema_version` | ✅ | 本仕様のバージョン (`1.0.0`) |
| `source_site` | ✅ | エクスポート元サイト URL |
| `exported_at` | ✅ | エクスポート時刻 (ISO 8601) |
| `user_achievements` | ✅ | UserAchievement 配列 |

トップに `handle_name` / `license` / `count` 等の冗長フィールドは置かない。

---

## 9.5 Quick Start (実装者向け 5 分ガイド)

### 9.5.1 JSON Schema で validate する

[ajv-cli](https://www.npmjs.com/package/ajv-cli) を使う例:

```bash
npm i -g ajv-cli ajv-formats
ajv validate \
  -s docs/spec/v1.0.0/achievement.schema.json \
  -d docs/spec/v1.0.0/examples/achievement.example.json \
  -c ajv-formats --spec=draft2020

ajv validate \
  -s docs/spec/v1.0.0/export.schema.json \
  -r docs/spec/v1.0.0/user_achievement.schema.json \
  -d docs/spec/v1.0.0/examples/export.example.json \
  -c ajv-formats --spec=draft2020
```

Python なら [jsonschema](https://pypi.org/project/jsonschema/):

```python
import json
from jsonschema import validate, Draft202012Validator

schema = json.load(open("docs/spec/v1.0.0/achievement.schema.json"))
data   = json.load(open("docs/spec/v1.0.0/examples/achievement.example.json"))
Draft202012Validator.check_schema(schema)
validate(instance=data, schema=schema)
```

**スキーマの URL について:**
- スキーマ内の `$id` は `https://ponta0321.github.io/game-achievement-common-spec/v1.0.0/...` を指す (GitHub Pages 有効化後に解決可能)
- 未解決でも validator はローカルファイルとして読めば動く (上の例)
- ネットワーク経由で取得したい場合の raw URL: `https://raw.githubusercontent.com/ponta0321/game-achievement-common-spec/main/docs/spec/v1.0.0/achievement.schema.json`

### 9.5.2 取り込み (import) の最小擬似コード

```pseudo
function import_export(file, current_user):
    payload = json_decode(file)

    # 1. schema_version は v1.x 範囲のみ受理 (前方互換)
    if not payload.schema_version.startswith("1."):
        return reject("unsupported schema_version")

    for record in payload.user_achievements:
        # 2. record_uid 重複チェック (再 import 防止)
        if exists_record(record.record_uid, current_user):
            continue  # skip silently

        # 3. achievement_uid + thumbnail.sha256 で識別
        ach_local = lookup_achievement(record.achievement_uid)
        if ach_local is None:
            # 自サイトに無い → snapshot から仮 Achievement を生成 or pending 投入
            ach_local = create_pending_from_snapshot(record.achievement_snapshot)

        # 4. item_refs を自サイトの item に紐付け (i18n タイトルも考慮)
        item = match_item(record.item_refs.external_ids)
                or match_item_by_title_platform(record.item_refs.title, record.item_refs.platform)
                or match_item_by_i18n_titles(record.item_refs.title_i18n, record.item_refs.platform)

        # 5. 表示用テキストはユーザー言語にフォールバック (§7.5.3)
        display_title = pick_i18n(
            record.achievement_snapshot.title_i18n,
            record.achievement_snapshot.title,
            user_lang = current_user.lang)

        # 6. pending で投入 (運営審査後に approved)
        insert_user_achievement(record, item, ach_local, status="pending")

    return ok()
```

### 9.5.3 エクスポート (export) の最小擬似コード

```pseudo
function export_for_user(user):
    records = []
    for ua in db.user_achievements.where(owner = user, status = "approved"):
        ach = db.achievements.find(ua.achievement_uid)
        records.append({
            record_uid:           ua.uid,
            achievement_uid:      ach.uid,
            achievement_snapshot: {
                title:            ach.title,
                title_i18n:       ach.title_i18n,            # 翻訳マップを丸ごと埋め込む
                description:      ach.description,
                description_i18n: ach.description_i18n,
                thumbnail:        ach.thumbnail,             # 16x16 のみ。thumbnails_extra は含めない
            },
            item_refs:            strip_internal_fields(ach.item_refs),  # local_id 等を除外、title_i18n は残す
            achieved_at:          ua.achieved_at,
            created_at:           ua.created_at,
            comment:              ua.comment,
            comment_lang:         ua.comment_lang,           # ユーザーが入力時に選んだ言語
            proof_url:            ua.proof_url,
        })
    return {
        schema_version:    "1.0.0",
        source_site:       "https://example.com/",
        exported_at:       iso_now(),
        user_achievements: records,
    }
```

---

## 10. 実装ガイド (実装者向け)

### 10.1 必須

- Achievement: `schema_version` を `"1.0.0"` 固定で付与
- `uid` / `record_uid` をグローバル一意に発行
- `primary_lang` を BCP 47 タグで明示 (§7.5)
- `item_refs.title` と `item_refs.platform` は最低限埋める
- Achievement の `license` を明示 (`CC0-1.0` 推奨)
- UserAchievement の `achievement_snapshot` を可能な限り埋める (取込み先サイトの表示品質に直結、i18n マップ含む)

### 10.1.1 platform 識別子

`item_refs.platform` は **大文字英数 + アンダースコア、2〜16 文字** とする。フリーテキスト (例: `"PlayStation"`, `"プレステ"`) は許容しない。

推奨値 (五十音/カテゴリ別):

| カテゴリ | 値 |
|---|---|
| PlayStation | `PS`, `PS2`, `PS3`, `PS4`, `PS5`, `PSP`, `PSV` |
| Nintendo (据置) | `FC`/`NES`, `SFC`/`SNES`, `N64`, `GC`, `WII`, `WIIU`, `NSW` |
| Nintendo (携帯) | `GB`, `GBC`, `GBA`, `NDS`, `3DS` |
| Xbox | `XBOX`, `X360`, `XBONE`, `XBSX` |
| その他 | `PC`, `ARC` (アーケード), `MD` (メガドライブ), `SAT` (サターン), `DC` (ドリームキャスト), `PCE` (PCエンジン), `NEOGEO` |

リージョン別命名 (FC/NES, SFC/SNES) は併存させる。日本市場主体のサイトは FC/SFC、欧米市場主体のサイトは NES/SNES を使うが、external_ids (Wikidata 等) で同一作品としてマッチ可能。

### 10.1.2 proof_url の推奨ホワイトリスト

`proof_url` は任意フィールドだが、SSRF / 偽装 URL / 短命ホスト対策として実装サイトはホワイトリストを設けること。推奨ホスト:

- 動画: `youtube.com`, `youtu.be`, `twitch.tv`, `nicovideo.jp`, `bilibili.com`
- 画像: `imgur.com`, `i.imgur.com`, `imgbb.com`, `gyazo.com`
- 配信アーカイブ: `twitcasting.tv`

短縮 URL (`bit.ly` 等) と任意ホスト (個人サーバ) は **拒否を推奨** (404 化リスク・改竄リスク)。

### 10.2 推奨

- `external_ids` (Wikidata / IGDB / MobyGames / RetroAchievements 等) を可能な限り埋める
  → サイト跨ぎの作品マッチング精度が劇的に上がる
- エクスポート時に内部運用フィールド (owner_code_hash / report_count 等) を **削除** する
- 日付フィールドは ISO 8601 で出力 (`achieved_at` は date、`created_at` は date-time)

### 10.3 拒否すべき入力 / 前方互換の扱い

**拒否すべき:**
- `schema_version` が v1.x 範囲外 (`^1\.` にマッチしない)
- `uid` / `record_uid` のフォーマット不正
- `item_refs.title` または `item_refs.platform` 欠落

**前方互換ルール (重要):**
- v1.0.0 実装は **v1.1+ の schema_version を持つデータを「受理」** すること (拒否しない)
- v1.x 系での MINOR バージョンアップは「任意フィールド追加」「enum 値追加」のみ → 未知フィールドは無視して読み飛ばすだけで動く
- JSON Schema レベルでは `additionalProperties: false` で締めているが、これは「v1.0.0 スキーマでの厳格検証用」。**実装の取り込みパスでは additionalProperties 違反は warning に留め reject しない** こと
- `external_ids` のみ `additionalProperties: true` で開いており、vendor 固有 ID (例: `steam_appid`) は MINOR バージョンを待たずに追加可能

### 10.4 改竄耐性 (取込み実装)

ユーザーがローカル端末で JSON を編集できる前提で、取込み側は以下を守る:

1. **uid だけを信用しない** — `achievement_uid` だけで判定すると偽装に脆弱
2. **画像 sha256 で照合** — `thumbnail.data_uri` のバイナリ sha256 を計算し、自サイトの「公式実績ハッシュテーブル」と突合 (一致すれば正規実績、不一致なら UGC 扱い)
3. **整合性が取れない入力は pending で投入** — 即拒否ではなく、運営者が後で判断できる状態にする
4. **取り込み元 (`source_site` / uid namespace) を保持** — 後追い検証や問い合わせのため
5. **proof_url のホストホワイトリスト適用** — §10.1.2 の推奨ホスト以外は受理しない or 警告

---

## 11. バージョニング規約

SemVer 厳守:
- **MAJOR** — 破壊的変更 (必須フィールド追加 / 型変更 / enum 値削除)。新ディレクトリ `docs/spec/v2.0.0/` を並行発行
- **MINOR** — 後方互換な追加 (任意フィールド追加 / enum 値追加)
- **PATCH** — 文言・例の修正のみ

v1.0.0 公開後、v1.x 系では **絶対に破壊的変更しない**。

**前方互換ポリシー (v1.0.0 実装者の義務):**
- v1.0.0 schema で validate しつつ、`schema_version` が `1.x` であれば受理する
- 未知のフィールドは無視する (`additionalProperties` 違反でも reject しない)
- `external_ids` の未知キーも保持する (将来 vendor 固有 ID が増える前提)

これにより v1.0.0 実装は v1.1+ のデータも壊さず読める。

---

## 12. v1.0.0 ドラフトの簡素化方針 (2026-06-25 改訂)

初期ドラフトから以下を撤去し、共通仕様としての無駄を排除した:

| 撤去項目 | 理由 |
|---|---|
| UserAchievement の `license` | 個々の達成は知的創作物でなく、ライセンスは Achievement に従う |
| UserAchievement の `handle_name` | `record_uid` の名前空間で発行元サイトは判明、誰のかは取込みコンテキストで管理 |
| UserAchievement の `verification` | v1.0.0 では `self_report` のみで運用、enum 拡張は v1.1 以降に検討 |
| UserAchievement の `schema_version` / `source_site` (レコード単位) | トップに1つあれば十分 |
| Thumbnail の `kind` / `preset_id` / `palette` / `license` / `attribution` | 識別は sha256 バイナリ照合に統一、preset_id によるヒントは改竄リスク高 |
| エクスポート JSON トップの `count` | `user_achievements.length` と冗長 |

これにより、共通仕様としてミニマルかつ自己完結したデータ構造に整理された。

---

## 13. 内部実装版との差分 (実装者向けメモ)

本仕様 (公開版) では削除しているが、サイト内部 DB で持つことが推奨されるフィールド:

| フィールド | 用途 |
|---|---|
| `owner_code_hash` | 編集権限の本人確認 |
| `report_count` | 通報集計 (モデレーション) |
| `moderation_status` | 運営による pending / approved / hidden 管理 |
| `local_db_id` | DB の PK (連番) |

エクスポート時に上記を除外することで、本仕様準拠の JSON を出力できる。

---

## 14. 参照実装

- ゲームソフトナビ (https://gamesoft-navi.com/) — 開発元・最初の実装
  - エクスポート: `/user_export.php`
  - インポート設計: `private/spec/import.md` (非公開)

---

## 15. 連絡先・改訂提案

- 改訂提案・バグ報告: [GitHub Issues](https://github.com/ponta0321/game-achievement-common-spec/issues)
- 仕様議論: [GitHub Discussions](https://github.com/ponta0321/game-achievement-common-spec/discussions) (有効化後)
- 手順詳細は [CONTRIBUTING.md](CONTRIBUTING.md) を参照

---

## 16. ロードマップ

| 時期 | マイルストーン |
|---|---|
| 2026-06 | 仕様 v1.0.0-draft GitHub 公開 (本ドキュメント)、reference 実装着手 |
| 2026-07 ~ 2026-12 | reference 実装 ([gamesoft-navi.com](https://gamesoft-navi.com/)) で MVP 運用、実データで検証 |
| 2026-12 (目標) | フィードバックを反映し **v1.0.0 として凍結** |
| v1.0.0 凍結後 | コミュニティからの提案で v1.1.0 (後方互換追加) / v2.0.0 (破壊的) を検討 |

**現在のステータス**: v1.0.0-draft。凍結前のため破壊的変更が起こりうる。本仕様で実装する場合は CHANGELOG を購読すること。
