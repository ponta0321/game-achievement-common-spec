# Conformance — v1.0.0

> Game Achievement Common Spec の 3 つの適合性レベル (Basic / Connected / Verified) と相互運用ルール。

---

## 1. 3 つの適合性レベル

| Level | 必須仕様 | 主な用途 | 想定実装者 |
|---|---|---|---|
| **Basic** | [Data](README.md) | JSON ファイルの import/export のみ | 個人開発者、小規模コミュニティサイト |
| **Connected** | Data + [Transport](transport.md) | OAuth 2 でサーバー間直接通信 | 中堅サイト、複数サイト連携を運営戦略にする事業者 |
| **Verified** | Data + Transport + [Trust](trust.md) | JWT 署名による真正性保証 | 大手・企業・GDPR 準拠要件のあるサービス |

各レベルは「上位互換」: Verified は Basic / Connected の機能を全て含む。Basic 同士・Connected 同士の通信も従来通り可能。

---

## 2. 適合性宣言

サイトは自サイトの適合性を `/.well-known/achievement-spec` で宣言する (詳細は [Transport spec §7](transport.md))。

```json
{
  "spec_version": "1.0.0",
  "conformance": "verified"
}
```

`conformance` の値:
- `"basic"` — Data spec のみ実装
- `"connected"` — Data + Transport
- `"verified"` — Data + Transport + Trust

`.well-known/achievement-spec` を持たないサイトは **Basic** とみなす (互換性のため)。

---

## 3. 相互運用マトリクス

送信側 → 受信側 で使うべきプロトコルとエンベロープ:

| 送信 \ 受信 | Basic | Connected | Verified |
|---|---|---|---|
| **Basic** | Data spec (export.schema.json) | Data spec ファイル import | Data spec ファイル import |
| **Connected** | Data spec ファイル fallback | Transport envelope (user_achievements) | Transport envelope (user_achievements) |
| **Verified** | Data spec ファイル fallback (署名は破棄) | Transport envelope (user_achievements_jws、受信側は署名無視可) | Transport envelope (user_achievements_jws、署名検証) |

ポイント:
- **Basic サイトは常に Data spec ファイル経由でしか参加できない** — オフライン交換は仕様の前提
- **直接通信は transport-envelope.schema.json** を使い、Connected は `user_achievements`、Verified は `user_achievements_jws` フィールドを返す (oneOf)
- **Verified 同士の通信は JWS 署名検証通過 → approved 即時投入可** (Transport spec §5.2)
- **混在ペア (Connected ↔ Verified) は JWS は付くが受信側が無視 → pending**
- **ファイル経由は常に pending** (Verified 自身からのファイル import でも署名は信頼しない、ユーザー介入の余地があるため)

---

## 4. 取り込み判定の推奨フロー

```pseudo
function on_receive(data, source_site):
    receiving_conformance = self.conformance       # basic / connected / verified
    sender_conformance    = fetch_well_known(source_site).conformance
    arrival_channel       = "oauth2" or "file"     # どこ経由で来たか

    case (sender_conformance, arrival_channel):

        ("verified", "oauth2"):
            jws = data.user_achievements_jws[*]
            if all_verify(jws, fetch_jwks(source_site)):
                return import(data, status="approved")  # 機械的真正性 OK
            else:
                return import(data, status="pending", reason="signature_failed")

        ("connected", "oauth2"):
            # OAuth 2 で sender 確実 → 軽い審査
            return import(data, status="pending", reason="auto_promote_eligible")

        ("verified", "file"):
            # 署名は付くが file 経由なので user 介入の可能性あり
            # JWS 検証は通るかもしれないが、ユーザーが古い JWS を貼り直した可能性
            # → pending 推奨、JWS exp で expired していたら reject
            return import(data, status="pending", reason="file_with_signature")

        ("basic", "file") or _:
            return import(data, status="pending", reason="standard_review")
```

---

## 5. 個人開発者の参加障壁ゼロ化

**Basic は永久に Basic でいてよい**。Verified サイトが普及しても Basic 実装は valid なまま。

- v1.x 系で Basic の必須要件は増えない
- JSON ファイルの import/export だけで「Game Achievement Common Spec 準拠」を宣言できる
- 採用判断時に「OAuth 2 や JWT を勉強する必要なし」

これにより仕様の **裾野** を維持しつつ、企業採用には Verified パスを提供する。

---

## 6. 企業採用の典型シナリオ

### シナリオ A: 既存大手 (PSN / Xbox 競合候補) の採用

1. 自サイトを Verified で実装
2. 自社の reference 実装 (例: gamesoft-navi.com) を Verified の連携先として OAuth 2 client 登録
3. ユーザーは「データを当社サービスに移行」をブラウザでクリック → OAuth 2 同意 → 自動取り込み
4. JWT 検証通過したレコードは即 approved → ユーザー体験ロスなし
5. 結果: ユーザーは Steam / Xbox の壁を超えて自社サービスに合流できる

### シナリオ B: GDPR データポータビリティ対応として採用

1. EU 圏の実績サイト運営者は GDPR Article 20 で「機械可読形式でユーザーデータを提供」義務あり
2. 本仕様 (Basic) の export 機能を実装するだけで Article 20 準拠の方法の 1 つを満たせる
3. さらに OAuth 2 (Connected) を実装することで「他サービスへの直接転送」も提供可能

### シナリオ C: コミュニティ運営者の段階的移行

1. v1 では Basic (JSON ファイル) のみ実装
2. ユーザー数が増えたら Connected に昇格、OAuth 2 連携先を開放
3. 企業連携を狙う段階で Verified に昇格、JWT 署名を導入

---

## 7. 「混在運用」の哲学的整合

§1 コア原則「**アンチ独占**」「**永続性**」との整合:

- **Verified が標準になっても Basic を排除しない** — Basic サイトとの JSON ファイル fallback で参加可能
- **小規模サイト消滅時、ユーザーは Verified サイトへ Basic 経由で逃げ込める** — 永続性の担保
- **大手が囲い込みのために Verified を独占する** ことは技術的に不可能 — 仕様は CC0、誰でも Verified サイトを立てられる

---

## 8. バージョニングと適合性

- Data / Transport / Trust の 3 spec は **独立してバージョニング** する
- 各 spec の version は `.well-known/achievement-spec` の以下で個別宣言:
  ```json
  {
    "data_spec_version": "1.0.0",
    "transport_spec_version": "1.0.0",
    "trust_spec_version": "1.0.0"
  }
  ```
- 例: Data v1.1 + Transport v1.0 + Trust v1.0 という組み合わせもあり得る
- いずれも MINOR 互換性ポリシーは Data spec §11 に準じる
