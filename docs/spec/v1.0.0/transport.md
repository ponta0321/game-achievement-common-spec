# Transport Spec — v1.0.0

> **Status**: 任意拡張 (Optional Layer 2)。本仕様は Data spec (achievement.schema.json / user_achievement.schema.json) を実装したサイト同士が **OAuth 2 経由でサーバー間直接通信** するためのプロトコルを定義する。
>
> Data spec のみ実装 (Basic conformance) のサイトは本仕様を実装する義務はない。

---

## 1. 目的

JSON ファイルのエクスポート/インポート (Data spec) はユーザー端末を経由するため、JSON が編集された可能性を取り込み側サイトが検出できない。

本仕様は以下を実現する:

- **ユーザー端末を経由しないサーバー間直接通信** — 機械的真正性の前提となる
- **OAuth 2.0 ベースの同意フロー** — GDPR Article 20 (データポータビリティ権) への直接対応
- **段階的採用** — 個人開発者は Basic のまま、企業は Connected / Verified へ拡張可能

---

## 2. 適合性レベル

サイトは自身の適合性を `.well-known/achievement-spec` で宣言する (§7)。

適合性レベル (Basic / Connected / Verified) の定義と相互運用ルールは [conformance.md](conformance.md) を Source of Truth とする。本仕様 (Transport spec) は **Connected 適合 (Data + 本仕様)** で必要となるエンドポイント・プロトコル・セキュリティ要件のみを定義する。

**重要**: Verified サイトは Basic サイトとも JSON ファイル経由で相互運用できる必要がある (後方互換)。

---

## 3. プロトコル要件

- **HTTPS 必須**。HTTP は禁止。TLS 1.2 以上
- **JSON のみ**。`Content-Type: application/json; charset=utf-8`
- **UTF-8 のみ**
- **エンドポイント URL は実装サイトが自由に決めてよい**。本仕様内の例は `/api/v1/oauth/authorize` 等の **推奨デフォルト** だが、サイトが別のパスを採用する場合は §7 の `/.well-known/achievement-spec` の `endpoints` フィールドで明示する。クライアントは well-known の値を絶対 URL として使うこと
- バージョニング: `Accept: application/vnd.achievement-spec.v1+json` で明示可。明示なき場合は v1 が応答

---

## 4. OAuth 2.0 フロー

[RFC 6749](https://www.rfc-editor.org/rfc/rfc6749) Authorization Code Flow + [RFC 7636](https://www.rfc-editor.org/rfc/rfc7636) PKCE を使用する。

### 4.1 ユースケース例 (サイト A → サイト B にユーザーデータを移す)

```
1. [サイト B] ユーザーがブラウザで「サイト A からデータをインポート」をクリック
2. [サイト B] サイト A の /api/v1/oauth/authorize へリダイレクト
   (client_id, redirect_uri, scope=achievements:read, state, code_challenge, code_challenge_method=S256)
3. [サイト A] ユーザーが「サイト B にこの範囲のデータを共有することに同意」
4. [サイト A] サイト B の redirect_uri に code を返す
5. [サイト B] サイト A の /api/v1/oauth/token に code + code_verifier を送り access_token を取得
6. [サイト B] サイト A の /api/v1/user_achievements?since=... に Bearer token で GET
7. [サイト A] transport-envelope.schema.json 準拠で UserAchievement[] (Connected) または JWS 配列 (Verified) を返す
8. [サイト B] 取り込み処理
```

### 4.2 必須エンドポイント

#### `GET /api/v1/oauth/authorize`

OAuth 2 認可エンドポイント。

クエリパラメータ:
- `client_id` — 取り込みサイトの登録済み ID
- `redirect_uri` — 登録済み URI
- `response_type=code`
- `scope` — `achievements:read` / `achievements:write` / `profile:read` のいずれか or 複数
- `state` — CSRF 対策
- `code_challenge`, `code_challenge_method=S256` — PKCE

#### `POST /api/v1/oauth/token`

token 交換。

リクエストボディ (`application/x-www-form-urlencoded`):
- `grant_type=authorization_code` / `refresh_token`
- `code` / `refresh_token`
- `code_verifier`
- `client_id`

レスポンス:
```json
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "...",
  "scope": "achievements:read"
}
```

### 4.3 必須スコープ

| Scope | 意味 | 主な用途 |
|---|---|---|
| `achievements:read` | UserAchievement の読み取り | サイト A から B へ取り込む際、B が A の `GET /api/v1/user_achievements` を呼ぶ |
| `achievements:write` | UserAchievement の追加 | サイト A から B への push 型移行。A が B の `POST /api/v1/user_achievements` を呼んでユーザーデータを送る (ユーザー先導の「データを B に転送」フロー) |
| `profile:read` | ユーザーのハンドル名等の基本プロファイル | 取り込み先で表示名を引き継ぎたい場合 |
| `offline_access` | refresh_token を取得 | 長期同期 (例: A の新規実績を B が継続取得) |

#### 4.3.1 read vs write の使い分け

- **Pull 型 (read)**: 取り込み元 (B) が起点。B が A の read scope を要求し、A からデータを取得する。OAuth 2 の主流パターン
- **Push 型 (write)**: 移行元 (A) が起点。A が B の write scope を要求し、A から B にデータを送る。サービス終了告知時の一括移行などに使われる

通常の「ユーザーが B サイトで A からのインポートをクリック」フローは Pull 型 (read) で十分。Push 型は A サイト側に「他サイトへの転送」UI を持つサービスのみが実装する。

---

## 5. データエンドポイント

### 5.1 `GET /api/v1/user_achievements`

ユーザーの達成記録を取得する。

クエリパラメータ:
- `since` (任意, ISO 8601) — `created_at` がこれ以降のものを取得
- `until` (任意, ISO 8601)
- `cursor` (任意) — pagination
- `limit` (任意, 1-1000, デフォルト 100)
- `lang` (任意, BCP 47) — 表示用言語ヒント (i18n マップから抽出)

レスポンスのスキーマは [transport-envelope.schema.json](transport-envelope.schema.json)。Connected サイトは `user_achievements` で UserAchievement オブジェクトを直接返し、Verified サイトは `user_achievements_jws` で JWS Compact 文字列を返す (両者は排他、oneOf)。

Connected サイトのレスポンス例:
```json
{
  "schema_version": "1.0.0",
  "source_site": "https://example.com/",
  "exported_at": "2026-06-26T12:00:00Z",
  "user_achievements": [ /* UserAchievement[] */ ],
  "pagination": {
    "next_cursor": "opaque_token",
    "has_more": true
  }
}
```

Verified サイトのレスポンス例:
```json
{
  "schema_version": "1.0.0",
  "source_site": "https://example.com/",
  "exported_at": "2026-06-26T12:00:00Z",
  "user_achievements_jws": [
    "eyJhbGc...header1.eyJpc3M...payload1.MEUCIQ...sig1",
    "eyJhbGc...header2.eyJpc3M...payload2.MEUCIQ...sig2"
  ],
  "pagination": { "next_cursor": null, "has_more": false }
}
```

**Data spec の export.schema.json (ファイル交換用) とは別スキーマ** であることに注意。`pagination` と `user_achievements_jws` は Transport envelope 固有のフィールド。

### 5.2 `POST /api/v1/user_achievements`

ユーザーの達成記録を受け取る (取り込み側エンドポイント、任意実装)。

リクエストボディ: `transport-envelope.schema.json` 形式 (Connected: `user_achievements`, Verified: `user_achievements_jws`)。

レスポンス:
```json
{
  "imported": 42,
  "skipped_duplicates": 3,
  "rejected": 1,
  "status": "pending"
}
```

`status` フィールドの値:
- `"approved"` — Verified サイトから JWS 付きで受信し、署名検証通過した場合 (機械的真正性 OK、即時投入)
- `"pending"` — それ以外 (Connected サイトからの直接通信、JWS 検証失敗、ファイル経由 import 等)
- `"rejected"` — schema 不整合、scope 不足、レート制限超過等

取り込み判定の完全フローは conformance.md §4 参照。

### 5.3 `GET /api/v1/achievements/{uid}`

特定の Achievement (定義) を取得する。サイト間で Achievement uid を解決するために使用。

レスポンス: achievement.schema.json 準拠 JSON。

---

## 6. レート制限・エラーハンドリング

### 6.1 レート制限

実装サイトは以下のヘッダで現状を返す:
- `X-RateLimit-Limit: 1000`
- `X-RateLimit-Remaining: 987`
- `X-RateLimit-Reset: 1730000000` (Unix epoch)

制限超過時は `429 Too Many Requests` + `Retry-After` ヘッダ。

### 6.2 エラーレスポンス

[RFC 7807](https://www.rfc-editor.org/rfc/rfc7807) Problem Details を使用する:

```json
{
  "type": "https://ponta0321.github.io/game-achievement-common-spec/errors/invalid_scope",
  "title": "Invalid scope",
  "status": 400,
  "detail": "The scope 'admin:*' is not supported.",
  "instance": "/api/v1/oauth/authorize?..."
}
```

### 6.3 標準エラーコード

| HTTP | type suffix | 意味 |
|---|---|---|
| 400 | `invalid_request` | パラメータ不正 |
| 401 | `invalid_token` | Bearer token 不正・期限切れ |
| 403 | `insufficient_scope` | scope 不足 |
| 404 | `not_found` | リソース無し |
| 409 | `conflict` | 一意制約違反 (例: 同一 client_id で別 redirect_uri を再登録) |
| 422 | `schema_validation_failed` | Data spec 準拠でない |
| 429 | `rate_limited` | レート制限 |
| 503 | `temporarily_unavailable` | メンテナンス等 |

**`record_uid` 重複の扱い**: import 処理中の `record_uid` 重複は **HTTP エラーにせず**、レスポンスボディの `skipped_duplicates` カウントで返す (§5.2)。バッチ取り込み中に一部レコードが重複していても、残りは正常に処理する。

---

## 7. Capability Discovery (`.well-known`)

[RFC 8615](https://www.rfc-editor.org/rfc/rfc8615) well-known URI 規約に従う。

### 7.1 `GET /.well-known/achievement-spec`

完全なサンプル: [examples/well-known.example.json](examples/well-known.example.json)

```json
{
  "spec_version": "1.0.0",
  "conformance": "verified",
  "data_spec_version": "1.0.0",
  "transport_spec_version": "1.0.0",
  "trust_spec_version": "1.0.0",
  "site_namespace": "example.com",
  "site_url": "https://example.com/",
  "endpoints": {
    "authorize": "https://example.com/api/v1/oauth/authorize",
    "token": "https://example.com/api/v1/oauth/token",
    "user_achievements": "https://example.com/api/v1/user_achievements",
    "achievements": "https://example.com/api/v1/achievements/{uid}"
  },
  "supported_scopes": [
    "achievements:read",
    "achievements:write",
    "profile:read",
    "offline_access"
  ],
  "supported_languages": ["ja", "en"],
  "import_policy": {
    "accepts_basic_json": true,
    "requires_verified_signature": false,
    "auto_approve_verified": true
  },
  "trust": {
    "jwks_uri": "https://example.com/.well-known/jwks.json",
    "jwks_archive_uri": "https://example.com/.well-known/jwks-archive.json",
    "issuer": "https://example.com/",
    "supported_algorithms": ["ES256", "EdDSA"]
  },
  "contact": "spec@example.com",
  "docs": "https://example.com/developers/achievement-spec/"
}
```

- `spec_version` は本仕様全体のメジャー (現状 `"1.0.0"` 固定)
- `data_spec_version` / `transport_spec_version` / `trust_spec_version` は **各 spec の独立バージョン** (conformance.md §8 参照)
- `conformance` が `basic` の場合、`endpoints` / `trust` フィールドは省略可

### 7.3 well-known の主要フィールド仕様

#### 7.3.1 `import_policy` オブジェクト

受信側サイトの取り込みポリシーを宣言する。送信側はこれを見て「相手がどう扱うか」を事前に把握できる。

| フィールド | 型 | 既定 | 意味 |
|---|---|---|---|
| `accepts_basic_json` | boolean | `true` | Basic サイトからの Data spec ファイル import を受け付けるか。`false` の場合、Verified 送信元のみ受理 |
| `requires_verified_signature` | boolean | `false` | `true` の場合、全 import に JWS 署名を要求 (Verified 同士の通信のみ受理)。Connected からの user_achievements は reject |
| `auto_approve_verified` | boolean | `false` | `true` の場合、Verified 経由 + JWS 検証通過のレコードを **approved 即時投入**。`false` の場合、検証通過後も pending 投入 (運営審査経由) |

組み合わせ例:
- 個人開発者向けオープン: `{accepts_basic_json: true, requires_verified_signature: false, auto_approve_verified: false}` — 何でも受け取って全 pending
- 企業向け閉鎖: `{accepts_basic_json: false, requires_verified_signature: true, auto_approve_verified: true}` — Verified 同士のみ即時 approve

#### 7.3.2 `supported_languages` 配列

サイトの **UI / 表示でサポートされる言語** (BCP 47 タグ) を宣言する。送信側はこれを見て、ユーザーに表示する言語を選択できる (`title_i18n` のうちどれが受信側で活きるか判定)。

- 値は BCP 47 タグの配列 (例: `["ja", "en", "zh-Hant"]`)
- 受信側がこの配列に無い言語のテキストを受け取った場合、UI 表示は primary_lang にフォールバックする
- 配列の **最初の要素が受信側のデフォルト UI 言語** (RFC 4647 の優先順)

このフィールドは「データの言語フィルタ」ではなく「表示の hint」。Data spec の primary_lang / title_i18n は引き続き全て保持されるべき (将来 UI 言語追加時のため)。

### 7.2 Capability discovery のフロー

```
[サイト B] サイト A からの import 開始
       ↓
[サイト B] GET https://a.com/.well-known/achievement-spec
       ↓
       conformance, supported_scopes, jwks_uri 等を解析
       ↓
       Verified 同士なら OAuth 2 + JWT 検証フロー
       Connected なら OAuth 2 のみ
       Basic なら「ユーザーに JSON ファイルダウンロードを案内」へフォールバック
```

---

## 8. セキュリティ要件

- **TLS 1.2 以上必須** (TLS 1.3 推奨)
- **client_secret** は安全に保管。Public Client (SPA / Mobile) は PKCE のみで client_secret 不使用
- **token revocation** ([RFC 7009](https://www.rfc-editor.org/rfc/rfc7009)) を実装推奨
- **CSRF**: state パラメータ必須
- **Open Redirect 防止**: redirect_uri は事前登録のみ許可
- **scope downgrade attack**: 要求した scope と発行された scope の検証
- **Bearer token は Authorization ヘッダのみ**。URL クエリへの埋め込み禁止

### 8.1 access_token の形式

- **Connected**: access_token の形式は実装サイト自由 (opaque token / JWT どちらでも可)
- **Verified**: access_token は **JWT 形式必須**。詳細は [Trust spec §6](trust.md) (Verified サイトの動作要件) を Source of Truth とする

Connected サイトが将来 Verified へ昇格する場合、access_token を opaque → JWT に変更することは互換性の観点で許される (クライアント側は token 値を opaque に扱うため)。

---

## 9. Data spec との関係

- Transport spec は **Data spec を完全に内包する**。やり取りされる UserAchievement / Achievement の JSON 構造は Data spec 準拠
- Transport spec を実装しないサイト (Basic) との相互運用は **JSON ファイル経由 (Data spec)** にフォールバック
- Data spec の `source_site` / `record_uid` namespace は Transport spec の `site_namespace` と一致する必要がある

---

## 10. 実装ガイド

### 10.1 Connected の最小実装

OAuth 2 server library (各言語にある) + REST エンドポイント 3 つだけ。一般的に 1-2 週間。

### 10.2 Verified の追加実装

JWT 署名 + JWKS 公開 + token 形式変更。Trust spec 参照。

### 10.3 OAuth 2 clients/users 管理 UI

各サイト運営者向けに「他サイトを連携先として登録」「自サイトに登録された他サイト一覧」「ユーザーの同意履歴」管理画面が必要。これは仕様外 (実装者責務)。
