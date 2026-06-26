# Trust Spec — v1.0.0

> **Status**: 任意拡張 (Optional Layer 3)。Transport spec の上に **JWT 署名による発行サイト真正性証明** を追加する。
>
> 本仕様を実装したサイトは Verified conformance を宣言できる。Connected (OAuth 2 のみ) サイトでも相互運用可能。

---

## 1. 目的

OAuth 2 ベースの直接通信 (Transport spec) では `Authorization: Bearer` の検証で「リクエスト元が認可済みクライアントである」ことは確認できるが、**応答 JSON 本体が確かにその発行サイトのサーバーで生成された** ことの暗号学的証拠は無い。

本仕様は以下を実現する:

- **発行サイトが UserAchievement / Achievement レコードに JWT 署名を付与**
- 受信サイトは公開鍵 (JWKS) で検証 → **真正性を機械的に確認できる**。検証通過時の取り込みポリシー (approved 即時 / pending) は受信サイトが well-known の `import_policy.auto_approve_verified` で宣言する
- **改竄不能性**: ユーザー介入を排除した OAuth 2 通信路 + 署名検証で機械的に真正性を確認

---

## 2. 採用されている標準

- [RFC 7515](https://www.rfc-editor.org/rfc/rfc7515) JSON Web Signature (JWS)
- [RFC 7517](https://www.rfc-editor.org/rfc/rfc7517) JSON Web Key (JWK / JWKS)
- [RFC 7518](https://www.rfc-editor.org/rfc/rfc7518) JSON Web Algorithms (JWA)

将来 [RFC 8414](https://www.rfc-editor.org/rfc/rfc8414) OAuth 2.0 Authorization Server Metadata との整合 (well-known の形式統一) を検討する予定だが、v1.0 では本仕様独自の `/.well-known/achievement-spec` 形式 (Transport spec §7) を使用する。

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

#### 重要: payload のシリアライズ規約

JWS は **payload バイト列に対する署名** であり、`base64url(payload)` 部分のバイト列がそのまま検証に使われる。受信側は **`base64url decode した bytes` を保持して使う** こと。一度 `json.loads` してから再 serialize すると、キー順序や空白の差で署名検証が壊れる。

送信側は payload JSON を **そのまま (任意のキー順、任意の空白で)** シリアライズして base64url 化してよい。受信側にはバイト列がそのまま到達するため、JCS (RFC 8785) 等の正規化は不要。

```python
# 送信側 (任意のキー順で OK)
payload_bytes = json.dumps(payload_dict, ensure_ascii=False).encode('utf-8')
jws = sign(header_bytes, payload_bytes, private_key)

# 受信側 (デコードした bytes を保持)
header_b64, payload_b64, sig_b64 = jws.split('.')
payload_bytes = base64url_decode(payload_b64)
if not verify(header_bytes, payload_bytes, sig_bytes, public_key):
    return reject
payload = json.loads(payload_bytes)  # 検証後に解析、再 serialize しない
data = payload['data']                # Data spec schema で validate
```

JWS library (jose / php-jwt / firebase/jwt 等) は通常この扱いを内部で行うため、library 経由なら問題ない。

#### header (例):
```json
{
  "alg": "ES256",
  "kid": "2026-key-1",
  "typ": "achievement-spec+jws",
  "cty": "application/json"
}
```

##### header フィールドの意味

| Claim | 必須 | 意味 | 活用 |
|---|---|---|---|
| `alg` | ✅ | 署名アルゴリズム (§3.2) | 検証時に使用 |
| `kid` | ✅ | 鍵 ID (jwks.json の `kid` と一致) | 鍵選択に使用 |
| `typ` | 🟡 | この JWS の type | 受信側が他形式 (汎用 JWT 等) と本仕様の JWS を区別するための識別子。値は **`achievement-spec+jws` 固定**。受信側は `typ != "achievement-spec+jws"` の JWS を本仕様外として無視してよい |
| `cty` | 🟡 | payload の Content-Type | `application/json` 固定。payload を JSON としてパースする hint |

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
| `exp` | 🟡 | 有効期限 (推奨)。本仕様での推奨上限は **5 年** (Code Signing 証明書の歴史的最大値、CA/Browser Forum BR の参考)。鍵漏洩時の影響範囲を抑制する目的。実装サイトはより短い期限を選んでよい |
| `kind` | ✅ | `uach` / `ach` / `export` |
| `data` | ✅ | Data spec 準拠 JSON オブジェクト |

### 4.3 署名付き形式の輸送

`transport-envelope.schema.json` は 3 つの相互排他形式 (oneOf) を持つ:

| 形式 | フィールド | 用途 |
|---|---|---|
| Plain (Connected) | `user_achievements` | UserAchievement 配列を未署名で返す |
| Per-record signed (Verified) | `user_achievements_jws` | UserAchievement ごとに JWS (`kind: "uach"`) を付与 |
| Bundle signed (Verified) | `bundle_jws` | export 全体に単一の JWS (`kind: "export"`) を付与 |

#### Per-record signed の例

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

各 JWS の `payload.kind = "uach"`、`payload.data` に UserAchievement (user_achievement.schema.json 準拠) が入る。

#### Bundle signed の例

```json
{
  "schema_version": "1.0.0",
  "source_site": "https://example.com/",
  "exported_at": "2026-06-26T12:00:00Z",
  "bundle_jws": "eyJhbGc...header.eyJpc3M...payload.MEUCIQ...sig",
  "pagination": { "next_cursor": null, "has_more": false }
}
```

`bundle_jws` の `payload.kind = "export"`、`payload.data` に Data spec の `export.schema.json` 形式 (`{schema_version, source_site, exported_at, user_achievements: [...]}`) が入る。1 リクエストで巨大なエクスポートを単一署名するシナリオ向け。

#### 使い分けの指針

- **Per-record (`user_achievements_jws`)**: ページネーション利用時、レコード単位の再利用が多い場合
- **Bundle (`bundle_jws`)**: エクスポート全体を一度に転送し、後で再パッケージしない場合 (署名が短く済む)
- 受信側はどちらも `transport-envelope.schema.json` の oneOf で判別可能

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
       5'. payload.data 内の uid namespace と iss の整合性を確認:
          - kind=uach の場合 record_uid と achievement_uid の両 namespace prefix
          - kind=ach  の場合 uid の namespace prefix
          - kind=export の場合 source_site と各 user_achievements[].record_uid namespace
          いずれかが iss の host と不一致 → reject (issuer_namespace_mismatch)
          ※ 中継・再署名は v1.0 では未対応 (v1.1 以降のクロス署名で検討)
       ↓
       6. payload.iat / exp の時刻整合性確認
       ↓
       7. 署名検証 (ES256 等)
       ↓
       8. payload.kind に応じて payload.data を validate:
          - "uach"   → user_achievement.schema.json
          - "ach"    → achievement.schema.json
          - "export" → export.schema.json
       ↓
       9. 検証通過 → 受信側ポリシーに応じて投入
          (auto_approve_verified=true なら approved、それ以外は pending)
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

### 6.1 鍵ローテーション履歴 (推奨)

過去の `kid` と公開鍵を長期検証用に保持する場合の推奨形式:

- **配置**: `/.well-known/achievement-spec` 内に `trust.jwks_archive_uri` フィールドで URL を示す (任意)
- **形式**: 現行 `jwks.json` と同じ JWKS 形式 (`{ "keys": [ ... ] }`)
- **保持期間**: 推奨 5 年 (`exp` 上限と整合し、Verified サイトの過去レコード検証が可能な期間)
- **Cache-Control**: 推奨 `max-age=86400` (24 時間、変更頻度は低い)

実装例:
```
/.well-known/jwks.json          ← 現行鍵 (検証で使う)
/.well-known/jwks-archive.json  ← 過去鍵 (長期検証で必要なときだけ取得)
```

このフィールドが無い場合、受信側は「現行 `jwks.json` で検証できない署名は無効」として扱う (`exp` 切れと同様)。

---

## 7. 信頼チェーンと多段経由 (重要)

v1.0 では「各サイトが自署名公開鍵を公開する」モデル (DNS / TLS と同じ Pin 信頼)。

### 7.1 多段経由データの扱い

A サイト → B サイト → C サイト と転送された UserAchievement について、C サイトから見た真正性は以下の通り:

- **JWS 署名の `iss` は常に発行サイト (A) を示す**
- B サイトが A サイトのレコードを C サイトに転送する場合、JWS 署名は **そのまま転送可能** (B が改変しない限り)
- ただし JWS の有効期限 (`exp`) と鍵ローテーション後の検証可能性は A サイトの jwks_uri 次第
- B サイトが Achievement の内容を編集して再配布する場合、**B サイトが新たに自署名する** (iss = B の URL)。その場合 C サイトから見ると「B サイトが保証する内容」となり、A サイトの真正性は失われる

### 7.2 v1.0 の前提

**v1.0 では「機械的真正性は1段経由までを保証」する**。多段経由データを Verified として扱いたい場合は、以下のいずれかが必要:

1. 中継サイト (B) が **原本の JWS をそのまま転送** (改変なし)
2. 受信側 (C) が A サイトの jwks_uri を直接取得して検証
3. (v1.1 以降の検討項目) クロス署名・Trust Registry・X.509 チェーン

### 7.3 将来 v1.1 以降での検討

- **コミュニティ運営の Trust Registry** — 信頼できるサイト一覧
- **クロス署名** — サイト同士が公開鍵に相互署名
- **証明書チェーン** — X.509 / Web of Trust
- **中継サイトの署名追加** — 元の JWS + 中継サイトの JWS の二重署名

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
