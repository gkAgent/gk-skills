# Spec Gen — TypeScript × MySQL

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-typescript-mysql` で自動起動します（貼り付け不要）。

---

以下の指示に従って、TypeScript × MySQL システムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。
> このスキルは TypeScript × MySQL / PlanetScale に特化しています。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> TypeScript × MySQL のプロジェクトを教えてください。
>
> - **フレームワーク**: Next.js / Express / NestJS / Hono / その他
> - **ORM/クライアント**: Prisma / Drizzle / mysql2 / Sequelize / 生SQL
> - **MySQL バリエーション**: MySQL 8.x / MariaDB / PlanetScale / TiDB

---

## Phase 1: 質問フェーズ

**Stage 1 — 基本情報**
1. システム名と対象機能は？
2. `schema.prisma` または `*.sql` / `drizzle/` は添付できますか？
3. 文字セット: `utf8mb4` / `utf8` / その他？

**Stage 2 — ORM 固有情報**
- Prisma: `provider = "mysql"` の確認、`relationMode = "prisma"` の有無（PlanetScale）
- Drizzle: `drizzle.config.ts` の `dialect: "mysql"`
- mysql2: `createConnection` / `createPool` の使い方

**Stage 3 — MySQL 固有情報**
- トランザクション方式（`BEGIN` / Prisma `$transaction`）
- `ON DUPLICATE KEY UPDATE` / `INSERT IGNORE` の使用有無
- PlanetScale の場合: Foreign Key制約なし設計の確認

---

## Phase 2: カバレッジプラン

全対象を列挙してから処理を開始してください。途中停止禁止。

---

## Phase 3: 仕様書生成 — TypeScript × MySQL テンプレート

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

## 絶対禁止事項

- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
- **PlanetScale FK制約欠如の見落とし禁止**: FK制約がないことを仕様書に明記する
