---
name: spec-gen-vbnet-postgresql
description: Generates detailed design specs (詳細仕様書) for VB.NET × PostgreSQL systems. Use when documenting VB.NET WinForms or WebForms applications connected to PostgreSQL via Npgsql, generating specs from VB.NET code using NpgsqlCommand or NpgsqlDataAdapter, or creating 詳細仕様書 for VB.NET × PostgreSQL systems migrated from Oracle/SQL Server. Covers Npgsql parameter naming, JSONB and UUID handling, NUMERIC precision with Decimal, TIMESTAMP with time zone semantics, and stored function (PL/pgSQL) calls. Extends spec-gen-enterprise-ja.
---

# Spec Generation — VB.NET × PostgreSQL

> Oracle / SQL Server から PostgreSQL に移行された VB.NET 案件、または新規に PostgreSQL を採用した VB.NET アプリの仕様書化に使う。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: Scope

Ask the user:

> 対象システムを教えてください。
>
> **VB.NET**: バージョン / プロジェクト種別（WinForms / WebForms / WPF / コンソール）
> **DB アクセス**: Npgsql（NpgsqlCommand）/ NpgsqlDataAdapter + DataSet / EF Core + Npgsql
> **PostgreSQL**: バージョン（12 / 14 / 16）/ ホスティング（自社 / RDS / Cloud SQL / Supabase）
> **規模**: フォーム数・モジュール数・テーブル数・関数（PL/pgSQL）数

---

## Phase 1: Question-Driven Setup

**Stage 1 — Basics**
1. システム名・機能名・主要フォーム名は？
2. `.vbproj` / `App.config` / `Web.config` は添付できますか？
3. `Option Strict On` が宣言されていますか？

**Stage 2 — VB.NET specifics**
- グローバル変数: `Module` に定義されている変数一覧
- エラー処理方式: `Try/Catch` / `On Error GoTo` / 混在
- イベントハンドラ: `Handles` 句 vs `AddHandler` の使い分け

**Stage 3 — PostgreSQL specifics**
- 接続文字列: `App.config` の `connectionStrings` セクション（`Host=...;Database=...`）
- パラメータ表記: `@param` か `:param` か（Npgsql は `@` を推奨）
- JSONB / UUID / 配列型カラムの利用有無
- PL/pgSQL 関数（`CREATE FUNCTION`）の呼び出し有無

---

## PostgreSQL 固有パターン（VB.NET 視点）

### Npgsql パラメータ表記（VB.NET）

```vbnet
' Npgsql は "@" 形式を推奨。":" は古い書式で互換のために残されている
Dim cmd As New NpgsqlCommand(
    "SELECT emp_id, emp_name FROM employee WHERE dept_id = @deptId AND active = @active", conn)
cmd.Parameters.AddWithValue("@deptId", deptId)
cmd.Parameters.AddWithValue("@active", True)
```

仕様書に記載すること:
- パラメータ表記スタイル（`@` / `:` のどちらを採用しているか）
- AddWithValue の使用箇所（型推論に依存する箇所として記録）

### NUMERIC 型 → VB.NET Decimal

```vbnet
' PostgreSQL NUMERIC(10,2) → VB.NET Decimal（Double/Single 使用禁止）
Dim salary As Decimal = Convert.ToDecimal(reader("salary"))
```

### TIMESTAMP WITH TIME ZONE（timestamptz）

```vbnet
' timestamptz は UTC 内部保持・クライアントタイムゾーンで返却される
' VB.NET 側は DateTimeOffset 受け取りを推奨
Dim createdAt As DateTimeOffset = CType(reader("created_at"), DateTimeOffset)

' DateTime で受けると Kind=Local になりタイムゾーン情報が失われる場合がある
```

仕様書に記載すること:
- カラム型が `timestamp` か `timestamptz` か
- VB.NET 側の受け取り型（`Date` / `DateTimeOffset`）

### JSONB カラムの扱い

```vbnet
' JSONB カラムは String として取得し JSON パーサで解釈
Dim jsonStr As String = reader("metadata").ToString()
' Newtonsoft.Json などで JObject に変換する想定

' 書き込み時は NpgsqlDbType.Jsonb を明示
cmd.Parameters.Add("@metadata", NpgsqlDbType.Jsonb).Value = jsonStr
```

### PL/pgSQL 関数呼び出し（VB.NET）

```vbnet
' PostgreSQL の関数呼び出しは SELECT 経由が基本
Dim cmd As New NpgsqlCommand(
    "SELECT * FROM fn_get_employee_by_dept(@deptId)", conn)
cmd.Parameters.AddWithValue("@deptId", deptId)

Using reader As NpgsqlDataReader = cmd.ExecuteReader()
    While reader.Read()
        ' 行集合を返す関数（RETURNS TABLE / SETOF）
    End While
End Using
```

---

## VB.NET × PostgreSQL 仕様書テンプレート

```markdown
## [フォーム名/モジュール名] — 詳細仕様

### 概要
| 項目 | 内容 |
|---|---|
| プロジェクト種別 | WinForms / WebForms |
| DB アクセス方式 | Npgsql 7.x |
| PostgreSQL バージョン | 16 |
| Option Strict | On / Off |

### SQL / 関数仕様
| 識別子 | 種別 | 対象テーブル/関数 | パラメータ |
|---|---|---|---|
| SELECT社員一覧 | SELECT | employee | dept_id: integer |
| fn_get_employee_by_dept | FUNCTION | （PL/pgSQL）| dept_id integer |

### 型マッピング
| PostgreSQL 型 | カラム | VB.NET 型 | 変換方法 |
|---|---|---|---|
| integer | emp_id | Integer | Convert.ToInt32 |
| numeric(10,2) | salary | Decimal | Convert.ToDecimal |
| timestamptz | created_at | DateTimeOffset | CType(..., DateTimeOffset) |
| jsonb | metadata | String → JObject | JSON.Parse |
| uuid | row_id | Guid | CType(..., Guid) |
| text[] | tags | String() | CType(..., String()) |

### Module/Shared グローバル変数
| 変数名 | 型 | 用途 | 変更タイミング |
|---|---|---|---|
```

---

## VB.NET 固有の追加確認事項

**Option Strict Off の場合（最重要）:**
- 暗黙的な型変換が多数存在する可能性 → 全て `TODO: 型変換の明示化` として記録
- レイトバインディング（`Object` 型への代入）→ 検出して全て列挙

**On Error GoTo の存在:**
- `On Error GoTo ErrorHandler` パターンが残っている箇所を全て列挙
- モダナイゼーション対象として記録

---

## Forbidden

- **NUMERIC → Double/Single 禁止** — 金額計算は Decimal を明記
- **timestamp / timestamptz の混同禁止** — 仕様書に必ず区別して記載
- **JSONB を String 単独で記載するのを避ける** — スキーマ（キー一覧）も別表で残す
- **AddWithValue のみ記載で型推論を放置しない** — `NpgsqlDbType` 明示の有無を仕様書に残す
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
