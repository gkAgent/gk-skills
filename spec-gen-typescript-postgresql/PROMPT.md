# Spec Gen — TypeScript × PostgreSQL

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-typescript-postgresql` で自動起動します（貼り付け不要）。

---

以下の指示に従って、TypeScript × PostgreSQL システムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。
> このスキルは TypeScript × PostgreSQL に特化しています。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> TypeScript × PostgreSQL のプロジェクトを教えてください。
>
> - **フレームワーク**: Next.js / React / NestJS / Express / Hono / その他
> - **ORM/クライアント**: Prisma / Drizzle / Kysely / node-postgres (pg) / 生SQL
> - **対象**: 画面 / API / バッチ / DB定義

---

## Phase 1: 質問フェーズ

**Stage 1 — 基本情報**
1. システム名と対象機能は？
2. `schema.prisma` / `*.sql` / `migrations/` は添付できますか？
3. 認証方式は？（NextAuth / Clerk / 自前 JWT）

**Stage 2 — TypeScript スタック**
- Node.js / Bun バージョン
- Prisma の場合: `generator` / `datasource` の設定
- Drizzle の場合: `drizzle.config.ts` の内容
- 生SQL の場合: パラメータバインド方式（`$1` / named）

**Stage 3 — PostgreSQL 固有情報**
- バージョン（14 / 15 / 16）
- スキーマ分離（public / tenant_id / Row Level Security）
- 拡張（pgvector / TimescaleDB / PostGIS 等）

---

## Phase 2: カバレッジプラン

ドキュメント化する全API / ページ / 関数を列挙してから処理を開始してください。

| # | 対象 | 種別 | ステータス |
|---|---|---|---|
| 1 | `GET /api/users` | API Route | [ ] |

---

## Phase 3: 仕様書生成 — TypeScript × PostgreSQL テンプレート

```markdown
# 詳細仕様書 — [機能名]

## 0. 基本情報
| 項目 | 内容 |
|---|---|
| フレームワーク | Next.js 14 / App Router |
| ORM | Prisma 5.x |
| DB | PostgreSQL 16 |

## 1. 処理概要

## 2. 入力
| パラメータ | 型 | 必須 | バリデーション |
|---|---|---|---|

## 3. DB操作
| 種別 | モデル/テーブル | 条件 | Prisma メソッド |
|---|---|---|---|
| SELECT | User | id = $1 | `findUnique` |

## 4. 非同期エラー処理
| エラー種別 | 発生箇所 | ハンドリング |
|---|---|---|
| `PrismaClientKnownRequestError` | DB操作 | P2025: 404返却 |

## 5. 出力

## 6. TODO / 要確認事項
```

---

## 絶対禁止事項

- **SQLインジェクション確認必須**: 生SQLを使う箇所では必ずパラメータバインド（`$1`）を明記。文字列連結SQLは仕様書に「セキュリティリスク: 要修正」と記載
- **型推測禁止**: `any`型の変数は「型不明 — 要確認」とTODOに落とす
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
