---
name: spec-gen-typescript-mysql
description: Generates design specification documents (詳細仕様書) from TypeScript source code using MySQL or PlanetScale. Use when documenting a Next.js/Express/NestJS application backed by MySQL, generating specs from TypeScript API code with Prisma/Drizzle/mysql2, or producing 詳細仕様書 for TypeScript × MySQL systems. Extends spec-gen-enterprise-ja. Covers Prisma MySQL quirks (no enum DDL rollback, unsigned int), Drizzle MySqlTable patterns, and mysql2 parameterized queries.
---

# Spec Gen — TypeScript × MySQL

> Full capability: install **spec-gen-enterprise-ja** for all 5 languages × 4 DBs.
> This skill focuses on TypeScript × MySQL / PlanetScale.

---

## Activation

Ask the user:

> TypeScript × MySQL のプロジェクトを教えてください。
>
> - **フレームワーク**: Next.js / Express / NestJS / Hono / その他
> - **ORM/クライアント**: Prisma / Drizzle / mysql2 / Sequelize / 生SQL
> - **MySQL バリエーション**: MySQL 8.x / MariaDB / PlanetScale / TiDB

---

## Phase 1: Question-Driven Setup

**Stage 1 — Basics**
1. システム名と対象機能は？
2. `schema.prisma` または `*.sql` / `drizzle/` は添付できますか？
3. 文字セット: `utf8mb4` / `utf8` / その他？

**Stage 2 — ORM specifics**
- Prisma: `provider = "mysql"` の確認、`relationMode = "prisma"` の有無（PlanetScale）
- Drizzle: `drizzle.config.ts` の `dialect: "mysql"`
- mysql2: `createConnection` / `createPool` の使い方

**Stage 3 — MySQL specifics**
- トランザクション方式（`BEGIN` / Prisma `$transaction`）
- `ON DUPLICATE KEY UPDATE` / `INSERT IGNORE` の使用有無
- PlanetScale の場合: Foreign Key制約なし設計の確認

---

## Phase 2: Coverage Plan

全対象を列挙してから処理開始。途中停止禁止。

---

## Phase 3: Spec Output — TypeScript × MySQL Template

```markdown
# 詳細仕様書 — [機能名]

## 0. 基本情報
| フレームワーク | ORM | DB |
|---|---|---|
| Next.js 14 | Prisma 5 | MySQL 8.0 |

## 3. DB操作
| 種別 | テーブル | 条件 | Prisma/SQL |
|---|---|---|---|
| SELECT | users | id = ? | `findUnique` |
| UPSERT | logs | - | `upsert` / ON DUPLICATE KEY |

## 4. MySQL固有注意事項
- `ENUM`型のカラム変更はマイグレーション時にロールバック不可（Prisma既知制限）
- `unsigned` INT は Prisma では `@db.UnsignedInt` として明示
- PlanetScale: Foreign Key制約なし → アプリ層での整合性担保が必要

## 5. TODO / 要確認事項
```

---

## Forbidden

- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
- **PlanetScale FK制約欠如の見落とし禁止**: FK制約がないことを仕様書に明記する
