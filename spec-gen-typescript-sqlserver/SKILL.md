---
name: spec-gen-typescript-sqlserver
description: Generates detailed design specs (詳細仕様書) for TypeScript × SQL Server systems — common in Azure-hosted Next.js and NestJS backends. Use when documenting TypeScript applications connected to SQL Server via Prisma, Drizzle, or mssql package, generating specs for Next.js API routes backed by SQL Server, or creating 詳細仕様書 for NestJS microservices using TypeORM × SQL Server. Covers SQL Server 2019/2022, Azure SQL Database, T-SQL specifics (TOP/OFFSET-FETCH, MERGE, OUTPUT clause), and Windows Authentication. Extends spec-gen-enterprise-ja.
---

# Spec Generation — TypeScript × SQL Server

> Azure / Microsoft 系バックエンドの定番: TypeScript × SQL Server。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: Scope

Ask the user:

> 対象システムを教えてください。
>
> **TypeScript**: フレームワーク（Next.js App Router / NestJS / Express / Hono）
> **ORM/クライアント**: Prisma / Drizzle / TypeORM / `mssql` / `tedious`
> **SQL Server**: バージョン（2019 / 2022 / Azure SQL）/ 認証（SQL Server 認証 / Windows 認証 / Azure AD）
> **規模**: APIエンドポイント数・モデル数・テーブル数

---

## SQL Server 固有パターン（TypeScript 視点）

### TOP vs OFFSET-FETCH ページネーション

```sql
-- SQL Server: TOP（件数制限のみ）
SELECT TOP 10 * FROM Employee ORDER BY EmpId;

-- SQL Server 2012+: OFFSET-FETCH（ページネーション）
SELECT * FROM Employee
ORDER BY EmpId
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;
```

```typescript
// Prisma: take/skip が自動的に OFFSET-FETCH に変換
const employees = await prisma.employee.findMany({
    skip: 20,
    take: 10,
    orderBy: { empId: "asc" },
});

// mssql 直接使用時はクエリを明示
```

### MERGE（UPSERT）

```typescript
// mssql / Prisma $queryRaw での MERGE
const result = await prisma.$queryRaw`
    MERGE INTO UserSettings AS target
    USING (VALUES (${userId}, ${key}, ${value})) AS src (UserId, Key, Value)
    ON target.UserId = src.UserId AND target.Key = src.Key
    WHEN MATCHED THEN UPDATE SET target.Value = src.Value
    WHEN NOT MATCHED THEN INSERT (UserId, Key, Value) VALUES (src.UserId, src.Key, src.Value);
`;
```

仕様書に記載すること: MERGE の MATCH 条件・WHEN MATCHED/NOT MATCHED の動作

### OUTPUT 句（挿入・更新後の値取得）

```typescript
// mssql: OUTPUT INSERTED.* で挿入後の全カラム取得
const result = await pool.request()
    .input("name", sql.NVarChar, name)
    .query("INSERT INTO Orders (Name) OUTPUT INSERTED.* VALUES (@name)");
const newOrder = result.recordset[0];
```

### IDENTITY 列と採番

```typescript
// Prisma schema: SQL Server の IDENTITY
model Employee {
  id    Int    @id @default(autoincrement())  // IDENTITY(1,1)
  name  String @db.NVarChar(100)
}
```

### nvarchar vs varchar（Unicode 対応）

```sql
-- 日本語を含むカラムは NVARCHAR を使用
-- VARCHAR は SQL Server のデフォルト照合順序では日本語を正しく扱えない場合あり
CREATE TABLE Employee (
    EmpName NVARCHAR(100) NOT NULL,  -- 日本語対応
    Code    VARCHAR(20)  NOT NULL    -- 英数字のみのコード類
);
```

Prisma での指定:
```prisma
model Employee {
  empName String @db.NVarChar(100)
  code    String @db.VarChar(20)
}
```

### Azure SQL 接続設定

```typescript
// Prisma の接続文字列（Azure SQL）
// DATABASE_URL="sqlserver://server.database.windows.net:1433;database=mydb;user=admin;password=xxx;encrypt=true;trustServerCertificate=false"

// mssql での Azure AD 認証
const config: sql.config = {
    server: "server.database.windows.net",
    authentication: {
        type: "azure-active-directory-default",  // Managed Identity
    },
    options: { encrypt: true, database: "mydb" },
};
```

---

## TypeScript × SQL Server 仕様書テンプレート

```markdown
## [ファイル名] — 詳細仕様

### 概要
| 項目 | 内容 |
|---|---|
| フレームワーク | Next.js 14 App Router / Prisma 5 |
| SQL Server | Azure SQL Database (2022 互換) |
| 認証 | Azure AD Managed Identity |

### APIエンドポイント
| メソッド | パス | 認証 | 処理概要 |
|---|---|---|---|
| GET | /api/employees | Bearer JWT | 社員一覧（ページネーション） |
| POST | /api/employees | Bearer JWT | 社員新規登録 |

### Prisma クエリ仕様
| 操作 | メソッド | WHERE条件 | 特記事項 |
|---|---|---|---|
| 一覧取得 | findMany | isActive=true | OFFSET-FETCH |
| UPSERT | $queryRaw | userId, key | MERGE 構文 |

### 型マッピング
| SQL Server 型 | Prisma 型 | TypeScript 型 | 備考 |
|---|---|---|---|
| NVARCHAR(100) | String @db.NVarChar(100) | string | 日本語対応 |
| DECIMAL(10,2) | Decimal | Decimal | prisma/client Decimal |
| BIT | Boolean | boolean | |
| DATETIME2 | DateTime | Date | |
```

---

## Forbidden

- **VARCHAR と NVARCHAR の区別を無記載禁止** — 日本語カラムは NVARCHAR を明記
- **MERGE 構文の MATCH 条件を無記載禁止** — UPSERT のキー条件を仕様書に記載
- **Prisma の Decimal 型を number で扱うコードを見落とし禁止** — 金融計算は Decimal 型を明記
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
