# 動画制作用情報ドキュメント

> **Game Achievement Common Spec v1.0.0** の発信動画 (解説動画風) を作るための情報をまとめたドキュメント。動画の演出・編集面は含まず、**素材としての情報** のみ。

---

## 0. 基本情報

| 項目 | 内容 |
|---|---|
| 仕様名 (正式) | Game Achievement Common Spec |
| 仕様名 (日本語) | ゲーム実績共通仕様 |
| バージョン | v1.0.0 |
| 公開日 | 2026-06-26 |
| ライセンス | CC0-1.0 (パブリックドメイン相当、商用利用可) |
| GitHub | https://github.com/ponta0321/game-achievement-common-spec |
| Release | https://github.com/ponta0321/game-achievement-common-spec/releases/tag/v1.0.0 |
| 公開仕様 URL (Schema $id) | https://ponta0321.github.io/game-achievement-common-spec/v1.0.0/ |
| reference 実装予定サイト | https://gamesoft-navi.com/ |

---

## 1. 動画タイトル候補

視聴者の関心を捉えるためのタイトル案を複数用意。ターゲット層に応じて選択。

### 一般ユーザー向け
- 「あなたの "プラチナトロフィー" 数千時間が、ある日消える話」
- 「Ubisoft が証明した "実績は永遠ではない" という現実」
- 「ゲーム実績を、サービス終了から守る方法」

### 開発者向け
- 「ゲーム実績を共通化するオープン仕様を作った話」
- 「Game Achievement Common Spec v1.0.0 公開のお知らせ」
- 「OAuth 2 + JSON Schema でゲーム実績を相互運用する」

### 採用検討者 (企業) 向け
- 「ゲーム業界の "データロックイン" を CC0 仕様で解決する」
- 「実績データのポータビリティ標準 v1.0.0」

---

## 2. 動画の核となるメッセージ

### キャッチコピー (1 行)
**「あなたのゲーム実績を、サービス終了から守る」**

### 1 文要約
> ゲームの達成記録 (アチーブメント) を、特定企業・特定サイトに依存しない形で共有・移転可能にするオープン標準仕様。

### 3 文要約
> 2024 年、Ubisoft の THE CREW がサービス終了し、プレイヤーの数千時間分の実績データが消滅した。同様の事態を防ぐため、ゲーム実績データを共通フォーマットで持ち運べるオープン仕様を策定・公開した。CC0 ライセンスで誰でも実装・採用可能、JSON Schema 準拠で技術的な敷居も低い。

---

## 3. 物語の骨格 (動画のストーリー構成)

解説動画として組み立てる際の流れ。各セクションに使える情報をまとめる。

### 3.1 オープニング: 問題提起

**訴求するシナリオ**:
- 「あなたが 3 年間プレイした FF14 の実績、いつまで残ると思いますか?」
- 「Steam のアカウントが消えたら? Microsoft が撤退したら?」
- 「実績データはあなたのものですか? それともサービス運営者のものですか?」

**実例として使えるサービス終了の歴史**:
- **THE CREW** (Ubisoft, 2014-2024 サービス終了) — 本仕様の動機の原点
- **Killzone** シリーズ (Sony, オンライン機能終了で実績の一部が達成不可に)
- **Wii Shop Channel** (Nintendo, 2019 終了)
- **Mixer** (Microsoft, 2020 終了)
- **Google Stadia** (2023 終了)

すべて「サービス運営者の判断ひとつでユーザーの積み上げが消える」事例。

### 3.2 問題の構造

**現状のゲーム実績エコシステムの問題**:

1. **vendor lock-in** — Steam の実績は Steam にしか残らない、PSN は PSN にしか
2. **データの所有権が運営者にある** — 退会・サービス終了で消える
3. **横断的な集計ができない** — 「私が今までゲームで達成した全実績」を一つに集められない
4. **後発プラットフォームが実装する標準がない** — 各社が独自仕様

### 3.3 解決のアイデア

「**サイトを跨いで持ち運べるデータ標準** を CC0 で公開する」

これにより:
- ユーザーがいつでも自分のデータをエクスポートできる
- 別サイトに移行できる
- サービス終了告知の 6 ヶ月前にバックアップ → 複数の代替サイトへ
- 任天堂以外のメインプラットフォーム (Steam / PSN / Xbox / RetroAchievements) からのデータ取り込みも可能

### 3.4 コア原則 (4 つ)

動画中で説明する仕様の哲学。それぞれ 1 例文でユーザー価値を語れる。

1. **データはユーザーに帰属** — どのサイトに記録しても本人が常にエクスポート/移転できる
2. **アンチ独占** — 単一サイト・単一企業に集約させない設計
3. **永続性** — 1 サイトが消滅しても他サイトでデータが活き続ける
4. **自己完結性** — 1 レコードだけ取り出しても何の作品の何の実績か表示可能

### 3.5 技術的概要 (簡潔に)

**3 層構成**:

| Layer | 適合性 | 主な機能 | 想定実装者 |
|---|---|---|---|
| Data spec | Basic | JSON 形式の import/export | 個人開発者・小規模サイト |
| Transport spec | Connected | OAuth 2 サーバー間通信 | 中堅サイト |
| Trust spec | Verified | JWT 署名による真正性証明 | 大手・企業 |

ポイント: **個人開発者は Basic だけ、企業は 3 層すべて**。段階的に採用可能。

### 3.6 デモ可能なシナリオ

動画中で実際に「動く」ところを見せられるシナリオ:

#### デモ 1: エクスポート
- A 社サイトで「データエクスポート」をクリック
- JSON ファイル (export.example.json) が生成される
- ファイルの中身を表示し、構造を解説

#### デモ 2: インポート
- B 社サイトで「データインポート」をクリック
- A 社の JSON ファイルをアップロード
- B 社の画面に取り込まれた実績が表示される

#### デモ 3: 大規模ユーザーの NDJSON
- 1 万件規模のヘビーユーザーが NDJSON 形式でエクスポート
- ストリーミング処理の様子

#### デモ 4: 多言語対応
- 日本語サイトで作った実績が、英語サイトで自動的に英訳タイトルで表示される

### 3.7 まとめ・行動喚起 (Call to Action)

- 仕様を見る: https://github.com/ponta0321/game-achievement-common-spec
- 採用・実装: CC0 なので誰でも自由に
- フィードバック: GitHub Issues / Discussions
- reference 実装: gamesoft-navi.com (今後対応予定)

---

## 4. 視聴者ペルソナ別の刺さるポイント

### 4.1 ゲームのヘビーユーザー (主要評価層)

**刺さる訴求**:
- 「あなたが Steam で 5,000 件達成した実績、移転できますか?」
- THE CREW の実例 (Ubisoft 2014-2024 終了で実績データ消失)
- 複数バックアップ可能、サイト消滅に強い

**強調すべき仕様の機能**:
- NDJSON 形式で大規模 export 対応
- 複数サイトへの分散バックアップ
- 公式プラットフォーム ID (Steam achievement_api_name 等) を保持

### 4.2 個人開発者

**刺さる訴求**:
- 「実績機能を作りたいけど一から仕様考えるのは大変」
- 「CC0 だから採用しやすい」
- 「JSON Schema + サンプルだけ追えば実装できる」
- 「自分のサイトと他サイトをつなげる夢が叶う」

**強調すべき仕様の機能**:
- Basic 適合は超軽量 (JSON ファイルの読み書きだけ)
- 公式 README §9.5 Quick Start に擬似コード完備
- ajv / Python jsonschema で validate 可能

### 4.3 中堅サービス運営者

**刺さる訴求**:
- 「GDPR データポータビリティ権対応に困っている」
- 「他社サービスとデータ連携したいけど毎回独自仕様作るのは無駄」
- 「ユーザーから "他社からデータ移したい" 要望がある」

**強調すべき仕様の機能**:
- OAuth 2 でユーザー駆動の連携
- 既存標準 (OAuth 2 / JSON Schema / RFC 7591) の組み合わせ
- 中規模実装で 1-2 週間程度

### 4.4 企業の事業開発者

**刺さる訴求**:
- 「ゲーム業界の vendor lock-in を打破するエコシステム作りに参加したい」
- 「JWT 署名による真正性保証で本格運用可能」
- 「CC0 で法務クリアランス最速」

**強調すべき仕様の機能**:
- Verified 適合で機械的真正性
- JWS / JWKS / 鍵ローテーション完全仕様
- 大手プラットフォーム ID 標準対応

### 4.5 一般視聴者 (ゲーマー)

**刺さる訴求**:
- 「ゲーム実績は誰のもの?」という哲学的問いかけ
- THE CREW の事例で危機感を共有
- 「あなたのプレイ時間は永遠じゃない」

**強調すべき仕様の機能**:
- ユーザー保護の哲学
- 複数バックアップ可能性
- 業界の変化への第一歩

---

## 5. 重要な事実・データ

### サービス終了の歴史 (動画の説得力強化用)

| 年 | サービス | 終了による影響 |
|---|---|---|
| 2014 | Microsoft 「Games for Windows Live」 | 多数の PC ゲームの実績システム停止 |
| 2019 | Nintendo Wii Shop Channel | デジタル所有ゲームの再 DL 不可 |
| 2020 | Mixer (Microsoft) | アカウントデータ移行困難 |
| 2023 | Google Stadia | 全ユーザーデータ消滅 |
| **2024** | **THE CREW (Ubisoft)** | **10 年運営のゲーム本体プレイ不可** |

### ゲーム実績システムの規模 (参考)

- Steam: 約 14 億アカウント、各ゲーム数十〜数百の実績
- PlayStation Network: 約 1 億アクティブユーザー、トロフィー数 1 億件超
- Xbox Live: 約 9000 万アクティブユーザー
- RetroAchievements: 約 50 万ユーザー、20 万件以上の実績

### 本仕様の規模 (情報の正確性のため)

- 仕様文書: 約 1500 行 (README + transport + trust + conformance)
- JSON Schema: 5 ファイル
- example: 5 ファイル (JSON + NDJSON)
- CC0-1.0 ライセンス、商用利用可、フォーク禁止条項なし

---

## 6. 解説に使える具体例

### 6.1 同じ実績が複数サイトで違う形で定義される例

| サイト | uid | title | thumbnail |
|---|---|---|---|
| A 社 (日) | `a-corp.com:ach:1234` | ナイツオブザラウンド入手 | A 社独自ドット絵 |
| B 社 (英) | `b-corp.com:ach:5678` | Got Knights of the Round | B 社独自ドット絵 |
| RetroAchievements | `retroachievements.org:ach:42` | Knights of the Round | RA 公式 64×64 |

→ これらを `external_ids` (公式 ID マッチング) または `aliases` (手動宣言) で統合可能

### 6.2 多言語表示の例

```json
{
  "primary_lang": "ja",
  "title": "ナイツオブザラウンド入手",
  "title_i18n": {
    "en": "Got Knights of the Round",
    "zh-Hant": "獲得圓桌騎士",
    "ko": "원탁의 기사 획득"
  }
}
```

- 日本人ユーザーが見ると: ナイツオブザラウンド入手
- 英語ユーザーが見ると: Got Knights of the Round
- 台湾繁体字ユーザーが見ると: 獲得圓桌騎士

### 6.3 サイト消滅時の救済シナリオ

```
[A 社] 2027 年 1 月 — サービス終了告知 (6 ヶ月後の 7 月終了)
   ↓
[a さん] 2027 年 2 月 — 自分の 500 件の実績を export (JSON or NDJSON)
   ↓
[a さん] 2027 年 3 月 — B 社・C 社の両方に同じデータを import
   ↓
[A 社] 2027 年 7 月 — サービス完全終了
   ↓
[a さん] 2027 年 8 月以降 — B 社と C 社で自分の実績データを継続利用
```

### 6.4 Steam からの取り込みシナリオ

a さんが Steam で 10,000 件の実績を持つヘビーユーザー:

```
[Steam] ISteamUserStats/GetPlayerAchievements API
   ↓
[変換ツール] Steam の (appid, achievement_api_name) を本仕様の
            Achievement.external_ids.steam_achievement_api_name に保存
   ↓
[NDJSON 形式] 10,000 件を 1 ファイルで export
   ↓
[本仕様準拠サイト] 取り込み → 機械的同一性判定で重複統合
```

---

## 7. 技術的に視覚化できる要素

動画で見せると分かりやすい部分。

### 7.1 図示できる構造

#### A. 3 層構成図

```
┌─────────────────────────────────┐
│         Trust Spec               │  ← Verified
│   (JWT 署名による真正性保証)        │
├─────────────────────────────────┤
│       Transport Spec             │  ← Connected
│   (OAuth 2 サーバー間通信)         │
├─────────────────────────────────┤
│          Data Spec               │  ← Basic
│   (JSON 形式の import/export)      │
└─────────────────────────────────┘
```

#### B. サイト消滅時の救済フロー

```
       A 社 (終了告知)
        │
        │ export
        ▼
    [JSON file]
       /  \
      ▼    ▼
   B 社    C 社  ← データが活き続ける
```

#### C. JSON Schema の関係図

```
export.schema.json (envelope)
        │
        └─ user_achievement.schema.json (1 件のレコード)
                │
                ├─ achievement_snapshot (内部)
                ├─ item_refs
                └─ achievement_uid → achievement.schema.json (参照)
```

### 7.2 コード例の見せ方

#### A. 最小の Achievement

```json
{
  "schema_version": "1.0.0",
  "uid": "example.com:ach:1234",
  "primary_lang": "ja",
  "title": "ナイツオブザラウンド入手",
  "item_refs": {
    "title": "ファイナルファンタジーVII",
    "platform": "PS"
  },
  "created_at": "2026-06-25T10:00:00+09:00",
  "license": "CC0-1.0"
}
```

#### B. 最小の UserAchievement

```json
{
  "record_uid": "example.com:uach:88012",
  "achievement_uid": "example.com:ach:1234",
  "item_refs": {
    "title": "ファイナルファンタジーVII",
    "platform": "PS"
  },
  "achieved_at": "2026-06-20",
  "created_at": "2026-06-20T22:30:00+09:00"
}
```

#### C. 検証コマンド

```bash
ajv validate \
  -s achievement.schema.json \
  -d achievement.example.json \
  -c ajv-formats --spec=draft2020
```

---

## 8. よくある質問への回答 (Q&A)

動画中またはコメント欄対応用。

### Q. なぜ既存の Steam Achievement / PSN Trophy ではダメなの?

A. それらは特定プラットフォーム専用で、Steam のデータを PSN に移すことはできません。本仕様はプラットフォーム中立で、誰でも実装・連携できます。

### Q. 既存サービスから取り込めるの?

A. データ自体は取り込めますが、各プラットフォームの利用規約で「データを第三者にエクスポートしてはいけない」と定められている場合があります。実装者は各規約を確認する必要があります。本仕様はあくまで「中立的なデータ形式」を提供します。

### Q. 1 社が独占的に採用したらどうなる?

A. CC0 ライセンスなので独占は技術的に不可能です。誰でも仕様を実装可能で、フォーク禁止条項もありません。

### Q. サムネ画像の著作権は?

A. 各サイトが用意するドット絵 (16×16) は実装者が著作権処理する責務です。本仕様自体は CC0 ですが、ゲームタイトル名・スクリーンショット等の権利関係は実装者が各法域で個別判断します。

### Q. なぜ任天堂対応がない?

A. 任天堂には公式のアカウント横断 Achievement システムが存在しないため、対応する公式 ID キーが定義できません。ただし、コミュニティが定義した実績については `aliases` フィールドで対応可能です。

### Q. 個人開発者でも実装できる?

A. はい。**Basic 適合** であれば JSON ファイルの読み書きだけで完結します。週末プロジェクトレベルで実装可能です。

### Q. v1.0.0 の後はどうなる?

A. v1.0.0 は **凍結** されています。v1.x 系で破壊的変更は行いません。任意フィールド追加などは v1.1.0 として、破壊的変更が必要な場合は v2.0.0 として並行発行します。

### Q. THE CREW のように、本仕様自体が消えたら?

A. 仕様は CC0 で GitHub に永続的に公開されており、誰でもフォーク・ミラー可能です。GitHub が消えても他のホスティングに移行可能です。仕様自身がアンチ独占の原則を体現しています。

---

## 9. ファイル素材一覧

動画で画面に映せる素材ファイル。

### 公開ファイル
- `README.md` (トップ) — 全体概要
- `docs/spec/v1.0.0/README.md` — 詳細仕様
- `docs/spec/v1.0.0/CHANGELOG.md` — 変更履歴
- `docs/spec/v1.0.0/CONTRIBUTING.md` — 貢献ガイド
- `docs/spec/v1.0.0/LICENSE` — CC0-1.0 全文
- `docs/spec/v1.0.0/transport.md` — OAuth 2 通信仕様
- `docs/spec/v1.0.0/trust.md` — JWT 署名仕様
- `docs/spec/v1.0.0/conformance.md` — 適合性レベル定義
- `docs/spec/v1.0.0/RELEASE_NOTES.md` — リリースノート

### JSON Schema
- `docs/spec/v1.0.0/achievement.schema.json`
- `docs/spec/v1.0.0/user_achievement.schema.json`
- `docs/spec/v1.0.0/export.schema.json`
- `docs/spec/v1.0.0/export-ndjson-meta.schema.json`
- `docs/spec/v1.0.0/transport-envelope.schema.json`

### サンプル
- `docs/spec/v1.0.0/examples/achievement.example.json`
- `docs/spec/v1.0.0/examples/user_achievement.example.json`
- `docs/spec/v1.0.0/examples/export.example.json`
- `docs/spec/v1.0.0/examples/export.example.ndjson`
- `docs/spec/v1.0.0/examples/well-known.example.json`

---

## 10. 動画作成チェックリスト

動画作成時に揃えるべき情報項目。

### 確認すべき事項
- [ ] タイトル候補から決定したか
- [ ] ターゲット視聴者を 1 つに絞ったか (一般ゲーマー / 個人開発者 / 企業)
- [ ] 動画の長さを決めたか (短編 3-5 分 / 標準 10-15 分 / 長編 20 分以上)
- [ ] 動画の主軸を決めたか (問題提起重視 / 技術解説重視 / デモ重視)
- [ ] 視聴者の感情のゴール (危機感 / 興奮 / 安心 / 行動意欲) を決めたか

### 説明すべき要素 (最低限)
- [ ] 仕様の名前と目的
- [ ] 公開日と現在のステータス (v1.0.0 凍結公開済み)
- [ ] CC0 ライセンス (商用可、フォーク自由)
- [ ] THE CREW などの動機となる事例
- [ ] コア原則 4 つ
- [ ] 3 層構成 (Basic / Connected / Verified)
- [ ] GitHub URL とリリース URL
- [ ] Call to Action (採用検討 / フィードバック)

### 補足すべき要素 (動画の質を上げるため)
- [ ] 公式プラットフォーム ID 対応 (Steam / PSN / Xbox / RetroAchievements)
- [ ] 多言語対応 (i18n マップ)
- [ ] NDJSON 形式の対応
- [ ] サイト消滅時の救済シナリオ
- [ ] 取り込みコンテキスト (誰のデータか) の設計

### 撮影/視覚素材
- [ ] GitHub リポジトリのスクリーンショット
- [ ] Release ページのスクリーンショット
- [ ] JSON Schema の構造図
- [ ] 3 層構成の図
- [ ] サンプル JSON のシンタックスハイライト
- [ ] THE CREW のサービス終了告知 (引用にあたる) または類似イメージ
- [ ] 検証コマンドの実行画面

### 関連情報の確認
- [ ] フォーク元 (gamesoft-navi の README) との関係性 (動画で触れるか)
- [ ] 任天堂対応がない理由を説明するか
- [ ] 企業採用での競合 (Steam / Xbox 等) との関係をどう語るか
- [ ] reference 実装の現状 (gamesoft-navi.com で対応予定) をどう説明するか

---

## 11. 追加情報・補足

### 仕様の差別化ポイント

既存の似たプロジェクトとの違い:

| 既存プロジェクト | 本仕様との違い |
|---|---|
| Steam Achievement API | 特定プラットフォーム専用、本仕様は中立 |
| ActivityPub | SNS 用、ゲーム実績特化はない |
| Open Badges (IMS Global) | 教育用バッジ、ゲーム実績の文脈と異なる |
| GitHub Achievements | GitHub 内専用 |

### 仕様の設計判断のポイント (深掘りトピック)

動画中で「なぜそうしたのか」を語る場合の根拠:

1. **CC0 ライセンスにした理由** — CC-BY-SA だと取り込み側の継承義務が「ユーザーが持ち運べる」原則と衝突するため
2. **改竄耐性をコア原則から外した理由** — sha256 識別の照合相手 (公式ハッシュレジストリ) が未定義で実装不能だったため、嘘を書かない選択
3. **3 層分離の理由** — 個人開発者を排除しないため。Basic だけで完結できる
4. **NDJSON 追加の理由** — ヘビーユーザー (主要評価層) の 10,000 件超のデータを単一 JSON で運ぶとブラウザがクラッシュする
5. **Achievement.external_ids 追加の理由** — Steam/PSN/Xbox 等から取り込んだ際に、公式 ID で機械的に同一性判定するため

### 検証実験の経緯 (信頼性アピール用)

動画で「ちゃんと動く仕様か検証した」を語る場合:

- Phase 1: 検証 1 〜検証 7 を実施
- 検証 1: 3 レイヤー (Basic/Connected/Verified) すべてで実装可能性確認
- 検証 2: A 社→B 社のユーザー実績移行シミュレーション
- 検証 3-7: Achievement 同等性、大規模 export、エラーパス、鍵ローテーション、サイト消滅時の救済
- 検出した P0 (実装不能級) 約 25 件をすべて修正

### 公開後の流れ (将来の展望)

- reference 実装 (gamesoft-navi.com) の対応状況は GitHub Discussions で公開予定
- コミュニティからのフィードバックで v1.1.0 / v2.0.0 検討
- v1.x 系では破壊的変更を行わない

---

## 12. 動画外の発信戦略 (参考)

動画と連動して発信できるチャネル。

### SNS 投稿用の短文

#### Twitter / X (140 文字)
> THE CREW のような事態でゲーム実績が消滅しないように。Game Achievement Common Spec v1.0.0 を CC0 で公開しました。Steam / PSN / Xbox 等から取り込み可、サイト間相互運用可。
> https://github.com/ponta0321/game-achievement-common-spec

#### Bluesky / Threads (250 文字)
> ゲーム実績データを特定企業に人質に取られない形で共有できる、オープン仕様 v1.0.0 を CC0 で公開しました。
> - 3 層構成で個人開発者から企業まで対応
> - 多言語対応・大規模ユーザー対応
> - Steam/PSN/Xbox 等の公式 ID 連携
> - サイト終了時のデータ救済シナリオ
> https://github.com/ponta0321/game-achievement-common-spec

#### Mastodon / Fediverse (500 文字)
> Game Achievement Common Spec v1.0.0 を公開しました。CC0 ライセンスで誰でも採用可能なゲーム実績共通仕様です。
> 2024 年 THE CREW のサービス終了でプレイヤーの数千時間が消滅した事例を踏まえ、ゲーム実績データをサイト間で持ち運べる標準を策定しました。
> - Basic 適合: JSON ファイルの読み書きだけ
> - Connected 適合: OAuth 2 でサーバー間直接通信
> - Verified 適合: JWT 署名で真正性保証
> Steam / PSN / Xbox / RetroAchievements の公式 ID にも対応。
> https://github.com/ponta0321/game-achievement-common-spec

### Qiita / Zenn 用の記事タイトル候補
- 「ゲーム実績を CC0 でオープン化する Game Achievement Common Spec v1.0.0 を公開しました」
- 「3 層構成 (Data / Transport / Trust) で設計したゲーム実績相互運用仕様」
- 「OAuth 2 + JWT でゲーム実績を真正性付きで持ち運ぶ仕様を作った」

### note 用 (より読み物寄り)
- 「サービス終了でゲーム実績が消える時代を終わらせる」
- 「Ubisoft の THE CREW から私たちが学ぶべきこと」

---

## 終わり

この情報があれば、ターゲット視聴者と動画の長さに応じて自由に構成できるはずです。不足や追加で必要な情報があれば適宜追記してください。
