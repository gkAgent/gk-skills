---
name: test-gen-vbnet-xunit
description: Generates Japanese-style unit test specifications (テスト仕様書) and xUnit.net test code stubs from VB.NET source. Use when creating 単体テスト仕様書 for VB.NET WinForms / WebForms / class libraries, generating xUnit test cases (.NET Framework 4.x or .NET 6+) for VB.NET projects, or producing test code that exercises Module-level state, Option Strict Off code paths, and On Error GoTo legacy error handling. Covers equivalence partitioning, boundary value analysis, mocking with Moq, and integration tests for ADO.NET DataAdapter patterns.
---

# Test Generation — VB.NET × xUnit.net

> VB.NET の単体テスト仕様書と xUnit.net テストコードを同時に生成する。
> WinForms / WebForms / クラスライブラリの全種に対応。
> 全言語版は **test-gen-enterprise-ja** を使うこと。

---

## Phase 0: Scope

Ask the user:

> テスト対象を教えてください。
>
> **対象**: クラス名 / モジュール名 / メソッド名
> **プロジェクト種別**: WinForms / WebForms / WPF / クラスライブラリ
> **.NET ターゲット**: .NET Framework 4.8 / .NET 6+ / .NET 8
> **xUnit バージョン**: xUnit.net 2.x（VB.NET から呼び出し）
> **モック**: Moq 4.x / 不要
> **成果物**: テスト仕様書のみ / テストコードのみ / 両方

---

## Phase 1: Question-Driven Setup

**Stage 1 — テスト対象の特定**
- 対象メソッドのシグネチャ（戻り値・引数）
- `Option Strict On / Off` の宣言状態
- `Module` グローバル変数への依存有無

**Stage 2 — テスト観点の洗い出し**
- 同値クラス（正常系の代表値・異常系の代表値）
- 境界値（下限-1, 下限, 下限+1, 上限-1, 上限, 上限+1）
- デシジョンテーブル（複数条件の組み合わせ）

**Stage 3 — 依存の切り離し**
- DB アクセスの有無（Mock / Stub / Testcontainers）
- 外部 API の有無（HttpClient / WebClient のモック）
- ファイル I/O の有無（一時ディレクトリ / メモリストリーム）

---

## VB.NET 固有のテスト観点

**Option Strict Off の場合（最重要）:**
- 暗黙的な型変換が起こる引数 → 正常型 + 異常型（String 渡し / Object 渡し）の両方をテスト
- レイトバインディング呼び出し → 存在しないメソッド呼び出しの異常系を必ず追加

**On Error GoTo の存在:**
- `On Error GoTo ErrorHandler` がある場合 → エラー誘発ケースを必ずテスト
- `Resume Next` がある場合 → エラー後の継続動作を仕様化

**Module グローバル変数への依存:**
- グローバル変数を読み取るメソッド → 各テスト前に明示的に初期化
- グローバル変数を書き換えるメソッド → テスト後にクリーンアップ

---

## テスト仕様書テンプレート（VB.NET 用）

```markdown
# 単体テスト仕様書

## テスト対象
| 項目 | 内容 |
|---|---|
| クラス/モジュール | DiscountModule |
| メソッド名 | CalculateDiscount(rank As String, amount As Decimal) As Decimal |
| Option Strict | Off |
| グローバル依存 | M_TaxRate（Module 変数） |
| 作成日 | YYYY-MM-DD |

## テストケース一覧

| No | 区分 | 入力 | 期待結果 | 観点 |
|---|---|---|---|---|
| T001 | 正常 | rank="Gold", amount=10000 | 1500 | Gold×以上→15% |
| T002 | 正常 | rank="Gold", amount=9999 | 999 | Gold×未満→10% |
| T003 | 境界値 | rank="Gold", amount=10000 | 1500 | 境界下限（以上） |
| T004 | 境界値 | rank="Gold", amount=9999 | 999 | 境界上限（未満） |
| T005 | 異常 | rank=Nothing, amount=10000 | NullReferenceException | Nothing 入力 |
| T006 | 異常 | rank="Gold", amount=-1 | ArgumentException | 負の金額 |
| T007 | 異常（型） | rank=123（Object 経由）, amount=10000 | InvalidCastException | Strict Off の型違反 |
| T008 | 状態 | M_TaxRate=0.1 で実行 | 1650 | グローバル状態反映 |
```

---

## テストコード生成（xUnit.net + VB.NET）

```vbnet
Imports Xunit
Imports Moq

Public Class DiscountModuleTests

    ' T001: Gold × 10,000円以上 → 15%
    <Fact>
    Public Sub CalculateDiscount_Gold_AmountAboveThreshold_Returns15Percent()
        Dim result As Decimal = DiscountModule.CalculateDiscount("Gold", 10000D)
        Assert.Equal(1500D, result)
    End Sub

    ' T005: Nothing ランク → NullReferenceException
    <Fact>
    Public Sub CalculateDiscount_NothingRank_ThrowsNullReferenceException()
        Assert.Throws(Of NullReferenceException)(
            Sub() DiscountModule.CalculateDiscount(Nothing, 10000D))
    End Sub

    ' T003/T004: 境界値（Theory + InlineData）
    <Theory>
    <InlineData("Gold", 9999, 999)>
    <InlineData("Gold", 10000, 1500)>
    Public Sub CalculateDiscount_Gold_BoundaryValues(
            rank As String, amount As Decimal, expected As Decimal)
        Dim result As Decimal = DiscountModule.CalculateDiscount(rank, amount)
        Assert.Equal(expected, result)
    End Sub

    ' T008: グローバル状態（M_TaxRate）反映
    <Fact>
    Public Sub CalculateDiscount_GlobalTaxRateApplied()
        Try
            GlobalState.M_TaxRate = 0.1D
            Dim result As Decimal = DiscountModule.CalculateDiscount("Gold", 10000D)
            Assert.Equal(1650D, result)
        Finally
            GlobalState.M_TaxRate = 0D  ' クリーンアップ
        End Try
    End Sub
End Class
```

---

## DB アクセスを含むテスト（VB.NET）

```vbnet
Public Class EmployeeRepositoryTests

    ' Moq で IDbConnection をモック化
    <Fact>
    Public Sub GetByDept_ReturnsEmployeeList()
        Dim mockConn = New Mock(Of IDbConnection)()
        ' ... モックセットアップ
        Dim sut As New EmployeeRepository(mockConn.Object)

        Dim result = sut.GetByDept(10)
        Assert.NotEmpty(result)
    End Sub
End Class
```

仕様書に記載すること:
- DB アクセス層の単体テストはモック / Testcontainers / 実DB のいずれを使うか
- 接続文字列の差し替え方法（`App.config` / 環境変数）

---

## Forbidden

- **テストケース番号の欠落禁止** — T001 から連番で全件記載
- **境界値の抜け禁止** — 下限-1 / 下限 / 下限+1 / 上限-1 / 上限 / 上限+1 を必ず含める
- **ハッピーパスのみ禁止** — 異常系（Nothing / 負値 / 型違反）を必ずセット
- **Module グローバル変数のクリーンアップ漏れ禁止** — `Try/Finally` または `IDisposable` で初期状態に戻す
- **Option Strict Off の異常系テスト省略禁止** — 型違反の入力ケースを必ず1件以上追加
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
