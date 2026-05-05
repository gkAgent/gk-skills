---
name: test-gen-csharp-xunit
description: Generates Japanese-style unit test specifications (テスト仕様書) and xUnit.net test code from C# source. Use when creating 単体テスト仕様書 for ASP.NET Core controllers/services, WPF view models, or C# class libraries, generating xUnit.net test code (.NET 6+ / .NET 8) with Moq mocking, or producing test code for async/await methods, EF Core repositories, and dependency-injected services. Covers equivalence partitioning, boundary value analysis, async exception handling, and FluentAssertions usage.
---

# Test Generation — C# × xUnit.net

> C# の単体テスト仕様書と xUnit.net テストコードを同時に生成する。
> ASP.NET Core / WPF / クラスライブラリの全種に対応。
> 全言語版は **test-gen-enterprise-ja** を使うこと。

---

## Phase 0: Scope

Ask the user:

> テスト対象を教えてください。
>
> **対象**: クラス名 / メソッド名 / コントローラ
> **プロジェクト種別**: ASP.NET Core MVC / Web API / Blazor / WPF / クラスライブラリ / コンソール
> **.NET ターゲット**: .NET 6 / .NET 8 / .NET Framework 4.8
> **テストフレームワーク**: xUnit.net 2.x
> **モック**: Moq 4.x / NSubstitute / 不要
> **アサーション**: 標準 Assert / FluentAssertions
> **成果物**: テスト仕様書のみ / テストコードのみ / 両方

---

## Phase 1: Question-Driven Setup

**Stage 1 — テスト対象の特定**
- 対象メソッドのシグネチャ（戻り値・引数）
- 同期 / 非同期（async / Task<T>）
- 依存注入されているサービス一覧

**Stage 2 — テスト観点の洗い出し**
- 同値クラス・境界値・デシジョンテーブル
- 例外系（ArgumentException / InvalidOperationException / カスタム例外）
- 非同期固有: キャンセル動作 / タイムアウト

**Stage 3 — 依存の切り離し**
- DB アクセス: EF Core In-Memory / SQLite / Testcontainers / Moq
- HTTP: HttpClient のモック（DelegatingHandler / IHttpClientFactory）
- ファイル I/O: System.IO.Abstractions / 一時ディレクトリ

---

## テスト仕様書テンプレート

```markdown
# 単体テスト仕様書

## テスト対象
| 項目 | 内容 |
|---|---|
| クラス | DiscountService |
| メソッド | CalculateDiscountAsync(string rank, decimal amount, CancellationToken ct) |
| 依存 | ITaxRateProvider |
| 戻り値 | Task<decimal> |
| 作成日 | YYYY-MM-DD |

## テストケース一覧

| No | 区分 | 入力 | 期待結果 | 観点 |
|---|---|---|---|---|
| T001 | 正常 | rank="Gold", amount=10000 | 1500 | Gold×以上→15% |
| T002 | 境界値 | rank="Gold", amount=10000 | 1500 | 境界下限（以上） |
| T003 | 境界値 | rank="Gold", amount=9999 | 999 | 境界上限（未満） |
| T004 | 異常 | rank=null, amount=10000 | ArgumentNullException | null 入力 |
| T005 | 異常 | rank="Gold", amount=-1 | ArgumentOutOfRangeException | 負の金額 |
| T006 | 非同期 | CT cancelled | OperationCanceledException | キャンセル動作 |
| T007 | 依存 | TaxRateProvider が例外 | InvalidOperationException | 依存層の例外伝播 |
```

---

## テストコード生成（xUnit + Moq + FluentAssertions）

```csharp
using Xunit;
using Moq;
using FluentAssertions;

public class DiscountServiceTests
{
    private readonly Mock<ITaxRateProvider> _taxProvider = new();
    private readonly DiscountService _sut;

    public DiscountServiceTests()
    {
        _taxProvider.Setup(x => x.GetAsync(It.IsAny<CancellationToken>()))
                    .ReturnsAsync(0.1m);
        _sut = new DiscountService(_taxProvider.Object);
    }

    // T001: Gold × 10,000円以上 → 15%
    [Fact]
    public async Task CalculateDiscountAsync_GoldAboveThreshold_Returns15Percent()
    {
        var result = await _sut.CalculateDiscountAsync("Gold", 10000m, CancellationToken.None);
        result.Should().Be(1500m);
    }

    // T002/T003: 境界値（Theory + InlineData）
    [Theory]
    [InlineData("Gold", 9999, 999)]
    [InlineData("Gold", 10000, 1500)]
    public async Task CalculateDiscountAsync_GoldBoundary(
        string rank, decimal amount, decimal expected)
    {
        var result = await _sut.CalculateDiscountAsync(rank, amount, CancellationToken.None);
        result.Should().Be(expected);
    }

    // T004: null ランク → ArgumentNullException
    [Fact]
    public async Task CalculateDiscountAsync_NullRank_ThrowsArgumentNullException()
    {
        Func<Task> act = () => _sut.CalculateDiscountAsync(null!, 10000m, CancellationToken.None);
        await act.Should().ThrowAsync<ArgumentNullException>();
    }

    // T006: キャンセル → OperationCanceledException
    [Fact]
    public async Task CalculateDiscountAsync_Cancelled_ThrowsOperationCanceledException()
    {
        using var cts = new CancellationTokenSource();
        cts.Cancel();
        Func<Task> act = () => _sut.CalculateDiscountAsync("Gold", 10000m, cts.Token);
        await act.Should().ThrowAsync<OperationCanceledException>();
    }

    // T007: 依存層の例外伝播
    [Fact]
    public async Task CalculateDiscountAsync_TaxProviderThrows_PropagatesException()
    {
        _taxProvider.Setup(x => x.GetAsync(It.IsAny<CancellationToken>()))
                    .ThrowsAsync(new InvalidOperationException("税率取得失敗"));
        Func<Task> act = () => _sut.CalculateDiscountAsync("Gold", 10000m, CancellationToken.None);
        await act.Should().ThrowAsync<InvalidOperationException>();
    }
}
```

---

## ASP.NET Core コントローラのテスト

```csharp
public class EmployeesControllerTests
{
    private readonly Mock<IEmployeeService> _service = new();
    private readonly EmployeesController _sut;

    public EmployeesControllerTests()
    {
        _sut = new EmployeesController(_service.Object);
    }

    [Fact]
    public async Task GetById_ExistingId_ReturnsOk()
    {
        _service.Setup(x => x.GetByIdAsync(1))
                .ReturnsAsync(new Employee { Id = 1, Name = "山田" });

        var result = await _sut.GetById(1);

        result.Should().BeOfType<OkObjectResult>()
              .Which.Value.Should().BeOfType<Employee>();
    }

    [Fact]
    public async Task GetById_NotFound_Returns404()
    {
        _service.Setup(x => x.GetByIdAsync(999))
                .ReturnsAsync((Employee?)null);

        var result = await _sut.GetById(999);

        result.Should().BeOfType<NotFoundResult>();
    }
}
```

---

## EF Core リポジトリのテスト

```markdown
## EF Core リポジトリのテスト方針

| アクセス先 | テスト方式 | 用途 |
|---|---|---|
| In-Memory Provider | Microsoft.EntityFrameworkCore.InMemory | 軽量・LINQ 動作の検証 |
| SQLite In-Memory | Microsoft.EntityFrameworkCore.Sqlite | 外部キー / トランザクション込みで検証 |
| Testcontainers | Testcontainers.PostgreSql 等 | 本番同等 DB で検証（推奨） |

注: In-Memory Provider はリレーショナル制約（外部キー / トランザクション挙動）を再現しないため、
DB 整合性が重要な箇所は SQLite or Testcontainers を選ぶ。
```

---

## Forbidden

- **テストケース番号の欠落禁止** — T001 から連番で全件記載
- **境界値の抜け禁止** — 下限/上限の前後を必ず含める
- **async メソッドの非 async テスト禁止** — `Task` 戻り値は必ず `await` する
- **キャンセル動作テストの省略禁止** — `CancellationToken` を受けるメソッドはキャンセル時動作を必ずテスト
- **依存層の例外伝播テスト省略禁止** — モックが投げた例外がそのまま外に出るかを確認
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
