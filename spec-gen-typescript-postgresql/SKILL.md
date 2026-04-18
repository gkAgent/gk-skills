---
name: spec-gen-typescript-postgresql
description: Generates detailed design specification documents (詳細仕様書) from TypeScript source code using PostgreSQL with Prisma, Drizzle, or raw SQL. Use when documenting a Next.js/React/NestJS application backed by PostgreSQL, generating specs from undocumented TypeScript API code, or producing 詳細仕様書 for a TypeScript full-stack system. Extends spec-gen-enterprise-ja with TypeScript × PostgreSQL focus. Covers async/await error handling, Prisma schema extraction, raw SQL parameterization safety.
---

# Spec Gen — TypeScript × PostgreSQL

> Full capability: install **spec-gen-enterprise-ja** for all 5 languages × 4 DBs.
> This skill focuses on TypeScript × PostgreSQL.

---

## Activation

Ask the user:

> TypeScript × PostgreSQL のプロジェクトを教えてください。
>
> - **フレームワーク**: Next.js / React / NestJS / Express / Hono / その他
> - **ORM/クライアント**: Prisma / Drizzle / Kysely / node-postgres (pg) / 生SQL
> - **対象**: 画面 / API / バッチ / DB定義

---

## Phase 1: Question-Driven Setup

Ask before generating anything.

**Stage 1 — Basics**
1. システム名と対象機能は？
2. `schema.prisma` / `*.sql` / `migrations/` は添付できますか？
3. 認証方式は？（NextAuth / Clerk / 自前 JWT）

**Stage 2 — TypeScript stack**
- Node.js / Bun バージョン
- Prisma の場合: `generator` / `datasource` の設定
- Drizzle の場合: `drizzle.config.ts` の内容
- 生SQL の場合: パラメータバインド方式（`$1` / named）

**Stage 3 — PostgreSQL specifics**
- バージョン（14 / 15 / 16）
- スキーマ分離（public / tenant_id / Row Level Security）
- 拡張（pgvector / TimescaleDB / PostGIS 等）

---

## Phase 2: Coverage Plan

List all API routes / pages / functions to document before starting.

| # | 対象 | 種別 | ステータス |
|---|---|---|---|
| 1 | `GET /api/users` | API Route | [ ] |

---

## Phase 3: Spec Output — TypeScript × PostgreSQL Template

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

## Forbidden

- **SQLインジェクション確認必須**: 生SQLを使う箇所では必ずパラメータバインド（`$1`）を明記。文字列連結SQLは仕様書に「セキュリティリスク: 要修正」と記載
- **型推測禁止**: `any`型の変数は「型不明 — 要確認」とTODOに落とす
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**（spec-gen-enterprise-jaの共通原則）
