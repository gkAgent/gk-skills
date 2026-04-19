# Spec Generation — C# × Oracle

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-csharp-oracle` で自動起動します（貼り付け不要）。

---

以下の指示に従って、C# × Oracle システムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

> 金融・製造業に多い C# × Oracle の組み合わせ。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 対象システムを教えてください。
>
> **C#**: .NET バージョン（Framework 4.x / .NET 6/7/8）/ フレームワーク（ASP.NET Core / WinForms / WPF / コンソール）
> **ORM**: ODP.NET 直接（OracleCommand）/ Dapper / EF Core（Oracle プロバイダ）
> **Oracle**: バージョン（11g / 12c / 19c）/ 接続方式（TNS / Easy Connect / JNDI）
> **規模**: クラス数・ストアドプロシージャ数・テーブル数

---

## Oracle 固有パターン（C# 視点）

### ODP.NET の OracleParameter バインド方式

```csharp
// ODP.NET: パラメータ名バインド（推奨）
using var cmd = new OracleCommand(
    "SELECT * FROM EMPLOYEE WHERE EMP_ID = :empId AND DEPT = :dept", conn);
cmd.Parameters.Add("empId", OracleDbType.Int32).Value = empId;
cmd.Parameters.Add("dept", OracleDbType.Varchar2).Value = dept;

// 注意: ODP.NET デフォルトは位置バインド。NamedParameters = true が必要
cmd.BindByName = true;  // ← 忘れると予期外の値バインドが発生
```

仕様書に記載すること:
- `BindByName` の設定状況
- パラメータの `OracleDbType` と C# 型のマッピング一覧

### NUMBER 型 → C# decimal

```csharp
// Oracle NUMBER(10,2) → C# decimal（BigDecimal 相当）
// double/float は精度損失が発生するため使用禁止
decimal salary = reader.GetDecimal(reader.GetOrdinal("SALARY"));

// EF Core × Oracle プロバイダ
[Column(TypeName = "NUMBER(10,2)")]
public decimal Salary { get; set; }
```

### Oracle DATE 型（時刻を含む）

```csharp
// Oracle DATE は時刻（HH:MM:SS）を含む
// C# DateTime にマップされるが、時刻部分の扱いを仕様書に明記すること
// 日付のみの比較が必要な場合は TRUNC() 使用

// OracleDataReader
DateTime hireDate = reader.GetDateTime(reader.GetOrdinal("HIRE_DATE"));
// 時刻部分が不要なら .Date プロパティで日付のみ取得
```

### ROWNUM ページネーション vs FETCH FIRST

```csharp
// Oracle 11g 以前: ROWNUM 方式
const string sql = @"
    SELECT * FROM (
        SELECT a.*, ROWNUM rn FROM (
            SELECT * FROM EMPLOYEE ORDER BY EMP_ID
        ) a WHERE ROWNUM <= :endRow
    ) WHERE rn > :startRow";

// Oracle 12c 以降: FETCH FIRST / OFFSET（EF Core でも自動生成）
// EF Core 使用時は Skip()/Take() が自動的に FETCH FIRST に変換される
var page = await _db.Employees
    .OrderBy(e => e.EmpId)
    .Skip(offset).Take(pageSize)
    .ToListAsync();
```

### ストアドプロシージャ呼び出し（ODP.NET）

```csharp
using var cmd = new OracleCommand("PKG_EMPLOYEE.GET_BY_DEPT", conn);
cmd.CommandType = CommandType.StoredProcedure;
cmd.Parameters.Add("p_dept_id", OracleDbType.Int32, ParameterDirection.Input).Value = deptId;
cmd.Parameters.Add("p_cursor", OracleDbType.RefCursor, ParameterDirection.Output);

// カーソル出力パラメータの仕様書記載必須
// - パッケージ名・プロシージャ名
// - IN/OUT パラメータ一覧と型
// - 返却カーソルのカラム定義
```

---

## EF Core × Oracle プロバイダ の仕様化

```csharp
// Oracle.EntityFrameworkCore プロバイダ設定
services.AddDbContext<AppDbContext>(opt =>
    opt.UseOracle(connectionString, o => o.UseOracleSQLCompatibility("19")));

// エンティティ設定の注意点
[Table("EMPLOYEE")]  // Oracle はデフォルト大文字テーブル名
public class Employee {
    [Column("EMP_ID")]
    public long EmpId { get; set; }  // Oracle NUMBER → long（主キー）

    [Column("HIRE_DATE")]
    public DateTime HireDate { get; set; }  // DATE型

    [Column("SALARY", TypeName = "NUMBER(12,2)")]
    public decimal Salary { get; set; }  // NUMBER → decimal 必須
}
```

---

## C# × Oracle 仕様書テンプレート

```markdown
## [クラス名] — 詳細仕様

### クラス概要
| 項目 | 内容 |
|---|---|
| パッケージ | com.example.repository |
| 役割 | [業務機能の説明] |
| フレームワーク | ASP.NET Core 8 / EF Core 8 (Oracle) |
| Oracle バージョン | 19c |

### SQL / ストアドプロシージャ仕様
| 識別子 | 種別 | 対象テーブル/パッケージ | パラメータ |
|---|---|---|---|
| GetByDept | SELECT | EMPLOYEE | dept_id: NUMBER |
| PKG_EMP.UPDATE | PROCEDURE | PKG_EMPLOYEE | emp_id, salary |

### 型マッピング一覧
| Oracle 型 | カラム名 | C# 型 | 備考 |
|---|---|---|---|
| NUMBER(6) | EMP_ID | long | PK |
| NUMBER(10,2) | SALARY | decimal | 精度保持 |
| DATE | HIRE_DATE | DateTime | 時刻含む |
| VARCHAR2(40) | EMP_NAME | string | |
```

---

## 絶対禁止事項

- **NUMBER → double/float 禁止** — 金融計算は decimal を明記
- **BindByName の記載漏れ禁止** — ODP.NET のパラメータバインド方式を必ず仕様書に記載
- **ROWNUM 方式と FETCH FIRST の区別を無記載禁止** — Oracle バージョンと対応するページネーション方式を明記
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
