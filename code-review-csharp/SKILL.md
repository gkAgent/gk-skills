---
name: code-review-csharp
description: Structured code review for C# .NET codebases (ASP.NET Core, .NET 6/7/8, .NET Framework 4.x). Use when reviewing ASP.NET Core Web API before production changes, auditing C# × SQL Server systems for security or quality, reviewing C# code inherited from a previous team, or assessing C# code before a .NET version upgrade. Extends code-review-enterprise-ja. Covers async void pitfalls, IDisposable/using statement leaks, EF Core N+1 queries, missing [Authorize] on controllers, SQL injection via string interpolation, and ConfigureAwait issues.
---

# Code Review — C# (.NET)

> Full coverage across 5 languages: install **code-review-enterprise-ja**.
> This skill focuses on C# — ASP.NET Core, EF Core, and .NET patterns.

---

## Activation

Ask: レビュー対象のクラス名と、プロジェクト種別（ASP.NET Core Web API / MVC / WinForms / WPF / Console）を教えてください。

---

## C# Review Checklist

### A. async/await の落とし穴（最重要）

- **`async void` メソッド** → 例外がキャッチされない（イベントハンドラ以外では使用禁止）
  ```csharp
  // 危険
  async void ProcessAsync() { await DoWork(); }  // 例外が消える

  // 正しい
  async Task ProcessAsync() { await DoWork(); }
  ```
- `.Result` / `.Wait()` の使用 → デッドロックの可能性（特に ASP.NET Core で）
- `ConfigureAwait(false)` の欠如 → ライブラリコードでは必須。アプリコードでは任意

### B. IDisposable / リソース管理

- `HttpClient` のインスタンス化を `new HttpClient()` でメソッド内に → ソケット枯渇
  ```csharp
  // 危険: 毎回 new HttpClient()
  using var client = new HttpClient(); // using は不要で有害（ソケット使い切り）

  // 正しい: IHttpClientFactory 注入 or static 共有
  private readonly HttpClient _client; // DI経由
  ```
- `DbContext` を `using` なしで使用（DI スコープ外で `new` している場合）
- `SqlConnection` / `SqlCommand` / `SqlDataReader` が `using` で囲まれているか
- `Stream` / `StreamReader` / `StreamWriter` の `using` 漏れ

### C. セキュリティ

- **SQL インジェクション**: 文字列補間による SQL 生成
  ```csharp
  // 危険
  var sql = $"SELECT * FROM Users WHERE Name = '{name}'";
  await _db.Database.ExecuteSqlRawAsync(sql);

  // 安全
  await _db.Database.ExecuteSqlInterpolatedAsync($"SELECT * FROM Users WHERE Name = {name}");
  // または
  await _db.Database.ExecuteSqlRawAsync("SELECT * FROM Users WHERE Name = {0}", name);
  ```
- `[Authorize]` の漏れ → コントローラ / アクションへの認可なしアクセス → 全件確認
- `ModelState.IsValid` チェック漏れ → バリデーション未実施でビジネスロジックに進む
- `TempData` / `ViewBag` にセンシティブ情報を格納 → セッションハイジャック時のリスク
- CORS: `AllowAnyOrigin().AllowAnyMethod().AllowAnyHeader()` → 全開放。理由を確認

### D. EF Core の N+1 問題

- LAZY ローディング（`UseLazyLoadingProxies()`）が有効で、ループ内に関連エンティティアクセス → N+1
  ```csharp
  // 危険
  var users = await _db.Users.ToListAsync();
  foreach (var user in users) { Console.WriteLine(user.Orders.Count); } // N+1

  // 対策
  var users = await _db.Users.Include(u => u.Orders).ToListAsync();
  ```
- `Select()` での射影なし全カラム取得 → 必要なカラムのみ `Select` で取得すること
- `AsNoTracking()` の欠如 → 読み取り専用クエリでのトラッキングは不要

### E. 例外処理

- `catch (Exception ex) {}` での飲み込み → 記録・修正
- `throw ex;` ではなく `throw;` を使っているか（スタックトレース保持）
- グローバル例外ハンドラ（`UseExceptionHandler` / `app.UseMiddleware<ExceptionMiddleware>`）の欠如 → スタックトレースがクライアントに漏れる

### F. 設定・シークレット管理

- `appsettings.json` への直接的な接続文字列 / APIキー記載 → 環境変数 / Azure Key Vault 推奨
- `IConfiguration` から直接パスワード取得（`_config["Db:Password"]`）→ 強く型付けされた `Options` パターン推奨
- `Development` 環境の設定が `Production` にも適用される構造 → 環境別設定ファイルの確認

---

## Review Report Output

```markdown
## レビュー結果: [クラス名]

### 重大（即修正）
- [行番号] 文字列補間による SQL 生成。ExecuteSqlInterpolatedAsync に変更せよ
- [行番号] async void メソッド。例外が消える。async Task に変更せよ

### 警告（修正推奨）
- [行番号] HttpClient を using でインスタンス化。ソケット枯渇の原因。IHttpClientFactory を注入せよ
- [行番号] EF Core: Orders の Include なし。N+1 クエリが発生する。Include(u => u.Orders) を追加せよ

### 情報
- [Authorize] 漏れが [N]件。認可チェックを追加することを推奨
- .Result 使用が [N]箇所。デッドロックリスクがある

### 総評
[断定口調で1〜3行]
```

---

## Modernization Assessment（モダナイゼーション判定）

| 項目 | リスク | 検出数 |
|---|---|---|
| `async void` | 高 | N |
| 文字列補間 SQL（SQL Injection） | 高 | N |
| `HttpClient` の都度 `new` | 高 | N |
| EF Core N+1（LAZY 未解決） | 中 | N |
| `.Result` / `.Wait()` | 中 | N |
| IDisposable `using` 漏れ | 中 | N |
| `[Authorize]` 漏れ | 高 | N |

**対象 .NET バージョン**: [.NET Framework 4.x / .NET Core 3.x / .NET 5+ / .NET 8]

---

## Forbidden

- **`async void` の見落とし禁止** — イベントハンドラ以外の全 `async void` を記録
- **`ExecuteSqlRaw` の文字列連結・補間を見落とし禁止** — SQLインジェクション直結
- **「〜の可能性があります」禁止** — 断定する
- **途中停止禁止** / **AI voice禁止**
