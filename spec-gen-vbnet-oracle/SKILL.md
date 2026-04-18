---
name: spec-gen-vbnet-oracle
description: Generates detailed design specs (詳細仕様書) for VB.NET × Oracle systems — legacy enterprise systems common in Japanese finance and government. Use when documenting VB.NET WinForms or WebForms applications connected to Oracle via ODP.NET, generating specs from VB.NET code using OracleCommand or OracleDataAdapter, or creating 詳細仕様書 for legacy VB.NET × Oracle 11g/19c systems before modernization. Covers ODP.NET BindByName, NUMBER precision with Decimal, DATE type, DataAdapter/DataSet patterns, and stored procedure calls. Extends spec-gen-enterprise-ja.
---

# Spec Generation — VB.NET × Oracle

> 金融・官公庁に多い VB.NET × Oracle の組み合わせ。レガシーシステムのドキュメント化に。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: Scope

Ask the user:

> 対象システムを教えてください。
>
> **VB.NET**: バージョン / プロジェクト種別（WinForms / WebForms / WPF / コンソール）
> **DB アクセス**: ODP.NET（OracleCommand）/ OracleDataAdapter + DataSet / ODBC / Oracle.ManagedDataAccess
> **Oracle**: バージョン（11g / 12c / 19c）
> **規模**: フォーム数・モジュール数・テーブル数

---

## Oracle 固有パターン（VB.NET 視点）

### ODP.NET の BindByName（VB.NET）

```vbnet
' BindByName = True を明示しないと位置バインドになる
Dim cmd As New OracleCommand(
    "SELECT * FROM EMPLOYEE WHERE EMP_ID = :empId AND DEPT = :dept", conn)
cmd.BindByName = True  ' ← 必須。False だと位置順でバインドされる
cmd.Parameters.Add("empId", OracleDbType.Int32).Value = empId
cmd.Parameters.Add("dept", OracleDbType.Varchar2).Value = dept
```

### DataAdapter + DataSet パターン（VB.NET レガシー）

```vbnet
' レガシーパターン: DataAdapter + DataSet
Dim da As New OracleDataAdapter(
    "SELECT EMP_ID, EMP_NAME, SALARY FROM EMPLOYEE WHERE DEPT_ID = :deptId", conn)
da.SelectCommand.Parameters.Add("deptId", OracleDbType.Int32).Value = deptId
Dim ds As New DataSet()
da.Fill(ds, "EMPLOYEE")

For Each row As DataRow In ds.Tables("EMPLOYEE").Rows
    Dim empName As String = row("EMP_NAME").ToString()
    Dim salary As Decimal = Convert.ToDecimal(row("SALARY"))  ' NUMBER → Decimal 必須
Next
```

仕様書に記載すること:
- DataSet の Fill 後に更新する場合の `OracleDataAdapter.UpdateCommand` の有無
- `DataRow` の値アクセスにおける型変換一覧

### NUMBER 型 → VB.NET Decimal

```vbnet
' Oracle NUMBER(10,2) → VB.NET Decimal（Double/Single 使用禁止）
Dim salary As Decimal = Convert.ToDecimal(reader("SALARY"))
' または
Dim salary As Decimal = CType(reader("SALARY"), Decimal)
```

### Oracle DATE 型（VB.NET）

```vbnet
' Oracle DATE は時刻（HH:MM:SS）を含む
' VB.NET の Date 型にマップされる
Dim hireDate As Date = CType(reader("HIRE_DATE"), Date)

' 日付のみ比較が必要な場合
Dim hireDateOnly As Date = hireDate.Date
```

### ストアドプロシージャ呼び出し（VB.NET）

```vbnet
Dim cmd As New OracleCommand("PKG_EMPLOYEE.GET_BY_DEPT", conn)
cmd.CommandType = CommandType.StoredProcedure
cmd.Parameters.Add("p_dept_id", OracleDbType.Int32, ParameterDirection.Input).Value = deptId
cmd.Parameters.Add("p_cursor", OracleDbType.RefCursor, ParameterDirection.Output)

Dim reader As OracleDataReader = cmd.ExecuteReader()
' REF CURSOR の処理
```

---

## VB.NET × Oracle 仕様書テンプレート

```markdown
## [フォーム名/モジュール名] — 詳細仕様

### 概要
| 項目 | 内容 |
|---|---|
| プロジェクト種別 | WinForms / WebForms |
| DB アクセス方式 | ODP.NET (Oracle.ManagedDataAccess) |
| Oracle バージョン | 19c |
| Option Strict | On / Off |

### SQL / ストアドプロシージャ仕様
| 識別子 | 種別 | 対象テーブル/パッケージ | パラメータ |
|---|---|---|---|
| SELECT社員一覧 | SELECT | EMPLOYEE | dept_id: NUMBER |
| PKG_EMP.UPDATE | PROCEDURE | PKG_EMPLOYEE | emp_id, salary |

### 型マッピング
| Oracle 型 | カラム | VB.NET 型 | 変換方法 |
|---|---|---|---|
| NUMBER(6) | EMP_ID | Integer/Long | CType / Convert.ToInt32 |
| NUMBER(10,2) | SALARY | Decimal | Convert.ToDecimal |
| DATE | HIRE_DATE | Date | CType(row("HIRE_DATE"), Date) |
| VARCHAR2(40) | EMP_NAME | String | .ToString() |
```

---

## Forbidden

- **NUMBER → Double/Single 禁止** — 金融計算は Decimal を明記
- **BindByName 記載漏れ禁止** — ODP.NET のバインド方式を必ず仕様書に記載
- **DataAdapter の UpdateCommand 有無を無記載禁止** — 更新処理がある場合は必ず記載
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
