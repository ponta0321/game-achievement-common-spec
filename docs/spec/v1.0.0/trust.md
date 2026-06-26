# Trust Spec — v1.0.0

> **Status**: 任意拡張 (Optional Layer 3)。Transport spec の上に **JWT 署名による発行サイト真正性証明** を追加する。
>
> 本仕様を実装したサイトは Verified conformance を宣言できる。Connected (OAuth 2 のみ) サイトでも相互運用可能。

---

## 1. 目的

OAuth 2 ベースの直接通信 (Transport spec) では `Authorization: Bearer` の検証で「リクエスト元が認可済みクライアントである」ことは確認できるが、**応答 JSON 本体が確かにその発行サイトのサーバーで生成された** ことの暗号学的証拠は無い。

本仕様は以下を実現する:

- **発行サイトが UserAchievement / Achievement レコードに JWT 署名を付与**
- 受信サイトは公開鍵 (JWKS) で検証 → 検証通過なら approved 即時投入可
- **改竄不能性**: ユーザー介入を排除した OAuth 2 通信路 + 署名検証で機械的に真正性を確認

---

## 2. 採用されている標準

- [RFC 7515](https://www.rfc-editor.org/rfc/rfc7515) JSON Web Signature (JWS)
- [RFC 7517](https://www.rfc-editor.org/rfc/rfc7517) JSON Web Key (JWK / JWKS)
- [RFC 7518](https://www.rfc-editor.org/rfc/rfc7518) JSON Web Algorithms (JWA)
- [RFC 8414](https://www.rfc-editor.org/rfc/rfc8414) OAuth 2.0 Authorization Server Metadata

---

## 3. 発行サイトの公開鍵公開

### 3.1 `GET /.well-known/jwks.json`

サイトの公開鍵セットを公開する:

```json
{
  "keys": [
    {
      "kty": "EC",
      "use": "sig",
      "crv": "P-256",
      "kid": "2026-key-1",
      "x": "...",
      "y": "...",
      "alg": "ES256"
    }
  ]
}
```

### 3.2 推奨アルゴリズム

| アルゴリズム | 推奨度 |
|---|---|
| `ES256` (ECDSA P-256 + SHA-256) | ✅ 推奨 (鍵長短い、検証高速) |
| `EdDSA` (Ed25519) | ✅ 推奨 (最新、シンプル) |
| `RS256` (RSA + SHA-256) | 🟡 互換性のため許容 (鍵長大きい) |
| `HS256` 等の対称鍵 | ❌ 禁止 (検証側に秘密鍵が必要なため公開検証不可) |

### 3.3 鍵ローテーション

- 同時に複数 `kid` を `jwks.json` に公開可
- 引退する鍵は `jwks.json` から削除する前に **最低 90 日間** は重複公開する (受信側のキャッシュ対応)
- 鍵漏洩時は即時削除 + 全 access_token revocation

---

## 4. レコード署名

### 4.1 署名対象

3 種類のオブジェクトに署名できる:

1. **UserAchievement 単体** — `kind: "uach"`
2. **Achievement 単体** — `kind: "ach"`
3. **Export バンドル** — `kind: "export"` (export.schema.json 全体)

### 4.2 JWS 構造

[RFC 7515](https://www.rfc-editor.org/rfc/rfc7515) Compact Serialization:

```
<base64url(header)>.<base64url(payload)>.<base64url(signature)>
```

#### header (例):
```json
{
  "alg": "ES256",
  "kid": "2026-key-1",
  "typ": "achievement-spec+jws",
  "cty": "application/json"
}
```

#### payload (UserAchievement 署名の場合):
```json
{
  "iss": "https://example.com/",
  "iat": 1730000000,
  "exp": 1761536000,
  "kind": "uach",
  "data": {
    "record_uid": "example.com:uach:88012",
    "achievement_uid": "example.com:ach:1234",
    "...": "..."
  }
}
```

| Claim | 必須 | 意味 |
|---|---|---|
| `iss` | ✅ | 発行サイト (= `site_url`) |
| `iat` | ✅ | 署名時刻 (Unix epoch) |
| `exp` | 🟡 | 有効期限 (推奨、最大 5 年) |
| `kind` | ✅ | `uach` / `ach` / `export` |
| `data` | ✅ | Data spec 準拠 JSON オブジェクト |

### 4.3 署名付き形式の輸送

#### 直接通信 (Transport spec) のレスポンスで返す場合

`transport-envelope.schema.json` を使用し、`user_achievements_jws` 配列で返す:

```json
{
  "schema_version": "1.0.0",
  "source_site": "https://example.com/",
  "exported_at": "2026-06-26T12:00:00Z",
  "user_achievements_jws": [
    "eyJhbGc...sig1",
    "eyJhbGc...sig2"
  ],
  "pagination": { "next_cursor": null, "has_more": false }
}
```

`user_achievements_jws` 配列の各要素が JWS 文字列 (Compact Serialization)。展開すると `payload.data` に UserAchievement が入っている。

**transport-envelope.schema.json は `user_achievements` と `user_achievements_jws` を排他**で扱う (oneOf)。Connected サイトは前者、Verified サイトは後者で返す。

#### JSON ファイル (Data spec) として保存する場合

Data spec の JSON Schema は `additionalProperties: false` で締めており、`x-signature` 等の独自フィールドを Data spec エンベロープに直接付加することは **できない**。

Verified サイトが署名付きエクスポートをファイルとして保存する場合は、以下のいずれかを選ぶ:

1. **Transport envelope 形式で保存**: `transport-envelope.schema.json` 準拠の JSON を `.json` ファイルに出力。Data spec の validator では検証できないが、Transport envelope validator で検証可
2. **Data spec 準拠でエクスポート (署名なし)**: Verified サイト間でもファイル経由の場合は署名を破棄。Basic 互換性最優先のシナリオ向け

詳細は conformance.md §3 の相互運用マトリクス参照。

---

## 5. 検証フロー

```
[サイト B] サイト A からの応答受信
       ↓
       1. JWS 形式パース (header / payload / signature 分解)
       ↓
       2. header.kid を取得
       ↓
       3. サイト A の jwks.json をキャッシュから取得 (なければ HTTP GET)
       ↓
       4. kid に一致する公開鍵を選択
       ↓
       5. payload.iss と jwks 取得元 origin が一致するか確認 (issuer 検証)
       ↓
       6. payload.iat / exp の時刻整合性確認
       ↓
       7. 署名検証 (ES256 等)
       ↓
       8. payload.data を Data spec の JSON Schema で validate
       ↓
       9. 検証通過 → approved 即時投入可
          検証失敗 → reject + ログ (signature_invalid / issuer_mismatch / expired 等)
```

### 5.1 キャッシュ規約

- jwks.json レスポンスの `Cache-Control` ヘッダを尊重 (デフォルト 24h)
- 検証失敗時は **1 回だけ** jwks 再取得を試みる (鍵ローテーション直後対策)
- 連続失敗時は警告ログ + pending 投入

---

## 6. Verified サイトの動作要件

Verified を宣言するサイトは以下を保証する:

| 要件 | 詳細 |
|---|---|
| 全 UserAchievement / Achievement への署名 | 自サイトから外向きに返す全レコードに JWS を付与 |
| OAuth 2 access_token も JWT 形式 | header に `alg`, `kid` を持ち、`/.well-known/jwks.json` の鍵で検証可 |
| `/.well-known/achievement-spec` 内に `trust.jwks_uri` を公開 | JSON 中の `trust.jwks_uri` フィールドで jwks.json の正規 URL を示す (Transport spec §7.1 参照) |
| 鍵ローテーション履歴の保持 | 過去 5 年の `kid` と公開鍵を `jwks-archive.json` で公開推奨 (長期検証用) |

---

## 7. 信頼チェーン (将来拡張)

v1.0 では「各サイトが自署名公開鍵を公開する」モデル (DNS / TLS と同じ Pin 信頼)。

将来 v1.1 以降での検討:
- **コミュニティ運営の Trust Registry** — 信頼できるサイト一覧
- **クロス署名** — サイト同士が公開鍵に相互署名
- **証明書チェーン** — X.509 / Web of Trust

これらは v1.0 では規定しない。

---

## 8. ユーザーが介入できない範囲の明示 (重要)

OAuth 2 + JWT 署名通信路を使う場合:

- **ユーザーがレコードを編集する余地は存在しない** — A → B サーバー間直接通信なので
- **A 側で生成された JWS は B 側で署名検証通過するまで信頼されない**
- ユーザーがブラウザ拡張で MITM することは TLS が防ぐ
- ローカルファイル経由 (Data spec) で同じ uid のレコードを持ち込まれた場合、JWS 無し → 取り込み側は pending 扱い (機械的真正性なし)

これが「**改竄耐性は本仕様の責務外 (Data spec §7.3)**」を **Trust spec が補完** する設計。

---

## 9. 個人開発者向けノート

Trust spec は重い。実装には:
- JWS library (各言語にある: `jose` / `python-jose` / `firebase/php-jwt` 等)
- 鍵管理 (HSM / KMS / 環境変数等)
- jwks.json ホスティング
- 鍵ローテーション運用

これら全部やりたくない個人開発者は **Connected (Transport spec のみ)** または **Basic (Data spec のみ)** に留めて OK。Verified サイトとの相互運用は引き続き可能 (Verified 側がフォールバックする)。

---

## 10. ライセンス

本仕様も CC0-1.0。
