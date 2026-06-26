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

| Level | 必須実装 | ファイル import/export | OAuth 2 通信 | JWT 署名 |
|---|---|---|---|---|
| **Basic** | Data spec | ✅ | — | — |
| **Connected** | Data spec + Transport spec | ✅ | ✅ | — |
| **Verified** | Data spec + Transport spec + Trust spec | ✅ | ✅ | ✅ |

**重要**: Verified サイトは Basic サイトとも JSON ファイル経由で相互運用できる必要がある (後方互換)。

---

## 3. プロトコル要件

- **HTTPS 必須**。HTTP は禁止。TLS 1.2 以上
- **JSON のみ**。`Content-Type: application/json; charset=utf-8`
- **UTF-8 のみ**
- 全エンドポイントは `/api/v1/` プレフィックス
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
7. [サイト A] UserAchievement[] を返す (Data spec 準拠 JSON)
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

| Scope | 意味 |
|---|---|
| `achievements:read` | UserAchievement の読み取り |
| `achievements:write` | UserAchievement の追加 (受信側エンドポイントを使う場合) |
| `profile:read` | ユーザーのハンドル名等の基本プロファイル |
| `offline_access` | refresh_token を取得 (任意) |

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

レスポンス:
```json
{
  "schema_version": "1.0.0",
  "source_site": "https://example.com/",
  "exported_at": "2026-06-26T12:00:00Z",
  "user_achievements": [ /* ... */ ],
  "pagination": {
    "next_cursor": "opaque_token",
    "has_more": true
  }
}
```

エンベロープは Data spec の export.schema.json と同一構造 + `pagination` キー追加。

### 5.2 `POST /api/v1/user_achievements`

ユーザーの達成記録を受け取る (取り込み側エンドポイント、任意実装)。

リクエストボディ: Data spec の export 形式と同一。

レスポンス:
```json
{
  "imported": 42,
  "skipped_duplicates": 3,
  "rejected": 1,
  "status": "pending"
}
```

すべての import は pending として受理し、運営審査後に approved にする (Data spec §10.4 と整合)。

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
| 409 | `conflict` | record_uid 重複等 |
| 422 | `schema_validation_failed` | Data spec 準拠でない |
| 429 | `rate_limited` | レート制限 |
| 503 | `temporarily_unavailable` | メンテナンス等 |

---

## 7. Capability Discovery (`.well-known`)

[RFC 8615](https://www.rfc-editor.org/rfc/rfc8615) well-known URI 規約に従う。

### 7.1 `GET /.well-known/achievement-spec`

```json
{
  "spec_version": "1.0.0",
  "conformance": "verified",
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
    "issuer": "https://example.com/"
  },
  "contact": "spec@example.com",
  "docs": "https://example.com/developers/achievement-spec/"
}
```

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
- **OAuth 2 access_token** は **JWT 形式** とすることを推奨 (Verified サイトでは必須、§Trust spec 参照)
- **CSRF**: state パラメータ必須
- **Open Redirect 防止**: redirect_uri は事前登録のみ許可
- **scope downgrade attack**: 要求した scope と発行された scope の検証
- **Bearer token は Authorization ヘッダのみ**。URL クエリへの埋め込み禁止

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
