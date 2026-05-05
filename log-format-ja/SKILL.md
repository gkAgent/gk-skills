---
name: log-format-ja
description: Designs Japanese-style application log formats (ログ設計書) for enterprise systems where logs are read by both ops staff and audit teams. Use when defining log levels, log message templates, structured logging fields, log rotation/retention policies, or producing ログ設計書 required by Japanese SI handover documents. Covers DEBUG/INFO/WARN/ERROR/FATAL hierarchy, Japanese message guidelines, correlation IDs, sensitive data masking (個人情報マスキング), JSON structured logs, and integration with NLog / log4net / Serilog / SLF4J / Python logging.
---

# Log Format Design — Japanese Enterprise Style

> 日本企業の業務システム向けにログ設計書を生成する。
> 運用担当者・監査担当者・開発者の三者が読めるフォーマットを定義する。

---

## Phase 0: Scope

Ask the user:

> ログ設計の対象を教えてください。
>
> **システム種別**: Web アプリ / バッチ / API / WinForms / 常駐プロセス
> **言語・フレームワーク**: Java / C# / VB.NET / TypeScript / Python / COBOL
> **ログライブラリ**: NLog / log4net / Serilog / SLF4J(Logback) / log4j2 / Python logging / 標準出力のみ
> **出力先**: ファイル / 標準出力（コンテナ） / Syslog / ログ集約基盤（ELK / Datadog / Splunk）
> **保管要件**: 保存期間（30日 / 1年 / 7年）/ 改ざん防止有無

---

## Phase 1: Question-Driven Setup

**Stage 1 — ログレベルの方針**
- DEBUG / INFO / WARN / ERROR / FATAL のどこまで使うか
- 本番出力レベル（INFO 以上 / WARN 以上）

**Stage 2 — メッセージ規約**
- 日本語メッセージか英語メッセージか（運用担当者の読みやすさ優先）
- メッセージID 採番ルール（例: `APP-E0001` `BATCH-W0042`）
- 例外スタックトレースの出力可否

**Stage 3 — 構造化ログ（推奨）**
- JSON 形式 / Key-Value 形式 / 単純テキスト
- 共通フィールド（timestamp / level / correlation_id / user_id 等）
- 個人情報・パスワード・カード番号のマスキング方針

---

## ログ設計書テンプレート

```markdown
# ログ設計書 — [システム名]

## 0. 基本情報
| 項目 | 内容 |
|---|---|
| 対象 | 受発注Webシステム |
| 言語 | C# / .NET 8 |
| ログライブラリ | Serilog 3.x |
| 出力先 | ファイル + Datadog |
| 保存期間 | アプリログ 90日 / 監査ログ 7年 |

## 1. ログレベル定義
| レベル | 用途 | 本番出力 | 例 |
|---|---|---|---|
| DEBUG | 開発者向けトレース | しない | リクエストパラメータ全件出力 |
| INFO | 業務操作の正常系 | する | ログイン成功、注文確定 |
| WARN | 異常ではないが注意 | する | リトライ発生、想定外の入力値（処理は継続） |
| ERROR | 業務エラー | する | バリデーションNG、外部API失敗 |
| FATAL | システム停止級 | する | DB接続喪失、致命的設定不備 |

## 2. メッセージID 採番ルール
| プレフィクス | 範囲 | 用途 |
|---|---|---|
| APP-Innnn | INFO | 業務正常系 |
| APP-Wnnnn | WARN | 警告 |
| APP-Ennnn | ERROR | 業務エラー |
| APP-Fnnnn | FATAL | 致命 |
| BATCH-* | バッチ専用 | 同上 |
| AUDIT-* | 監査ログ | 認証・権限変更・データ変更 |

## 3. 共通フィールド（JSON 構造化ログ）
| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| timestamp | ISO 8601 | ✅ | `2026-05-02T10:15:30+09:00` |
| level | string | ✅ | INFO/WARN/ERROR/FATAL |
| message_id | string | ✅ | APP-E0042 |
| message | string | ✅ | 日本語メッセージ |
| correlation_id | string | ✅ | リクエスト追跡用 UUID |
| user_id | string | 認証後 | ハッシュ化推奨 |
| ip_address | string | Webのみ | クライアントIP |
| service | string | ✅ | サービス名（ERPコア / 受注API 等） |
| host | string | ✅ | サーバホスト名 |
| stack_trace | string | ERROR以上 | 例外スタック |

## 4. 出力例（JSON）
```json
{
  "timestamp": "2026-05-02T10:15:30+09:00",
  "level": "ERROR",
  "message_id": "APP-E0042",
  "message": "注文登録に失敗しました（在庫不足）",
  "correlation_id": "f7a1b2c3-d4e5-6789-abcd-ef0123456789",
  "user_id": "u_8a3f...（hashed）",
  "service": "order-api",
  "host": "app-srv-01",
  "context": {
    "order_id": "O-20260502-00123",
    "product_code": "PRD-9988"
  }
}
```

## 5. マスキング対象（個人情報保護）
| データ種別 | マスキング方法 |
|---|---|
| パスワード | 完全削除（フィールド自体を出さない） |
| クレジットカード番号 | 下4桁のみ（`**** **** **** 1234`） |
| マイナンバー | 完全削除 |
| メールアドレス | 局所部1文字+ドメイン（`y***@example.com`） |
| 電話番号 | 末尾4桁のみ |
| 氏名 | 監査ログ以外は user_id に置き換え |

## 6. ローテーション・保管
| 項目 | 設定 |
|---|---|
| ローテーション | 日次 + 100MB 超過 |
| 圧縮 | gzip（翌日以降） |
| 保管期間 | アプリログ 90日 / 監査ログ 7年 |
| 退避先 | S3 / Azure Blob（長期はGlacier層）|
| 削除方式 | バッチ削除（cron） |

## 7. ログ出力禁止事項
- リクエストボディの全文出力（個人情報混入リスク）
- SQL 文のパラメータ展開後の値出力（個人情報混入リスク）
- スタックトレースの本番 INFO 出力（DEBUGのみ）
- 認証トークン / API キー / 秘密鍵の値
```

---

## 言語別 ログ出力サンプル

### C# (Serilog)

```csharp
Log.Information("APP-I0001 注文を受け付けました OrderId={OrderId} UserId={UserId}",
    order.Id, MaskUserId(user.Id));

Log.Error(ex, "APP-E0042 注文登録に失敗しました OrderId={OrderId}", order.Id);
```

### Java (SLF4J + Logback)

```java
log.info("APP-I0001 注文を受け付けました OrderId={} UserId={}",
    order.getId(), maskUserId(user.getId()));

log.error("APP-E0042 注文登録に失敗しました OrderId={}", order.getId(), ex);
```

### VB.NET (NLog)

```vbnet
logger.Info("APP-I0001 注文を受け付けました OrderId={0} UserId={1}",
    order.Id, MaskUserId(user.Id))

logger.Error(ex, "APP-E0042 注文登録に失敗しました OrderId={0}", order.Id)
```

### Python (logging)

```python
logger.info("APP-I0001 注文を受け付けました OrderId=%s UserId=%s",
            order.id, mask_user_id(user.id))

logger.exception("APP-E0042 注文登録に失敗しました OrderId=%s", order.id)
```

---

## 監査ログ（AUDIT-*）の設計指針

監査ログは別ファイル / 別ストリームに出すのが望ましい。

| 観点 | 設計指針 |
|---|---|
| 出力対象 | 認証成功/失敗、権限変更、マスタデータ更新、エクスポート操作 |
| 改ざん防止 | 追記のみ（write-once）/ ハッシュチェーン / WORM ストレージ |
| 保管期間 | 7年（電帳法・個人情報保護法を満たす想定） |
| 必須フィールド | who（user_id）/ when（timestamp）/ what（操作）/ where（IP）/ result（成功/失敗） |

---

## Forbidden

- **個人情報の生出力禁止** — マスキング規約を必ず定義
- **メッセージID 未採番禁止** — 全ログ出力にメッセージIDを付与する設計を必ず作る
- **スタックトレースの本番 INFO 出力禁止** — ERROR 以上のみ
- **保管期間の未定義禁止** — 法令要件（電帳法 / 個人情報保護法）を確認した上で記載
- **DEBUG 本番出力前提の設計禁止** — DEBUG は本番で抑止する前提
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
