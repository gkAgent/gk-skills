# コードレビュー — C# (.NET)

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/code-review-csharp` で自動起動します（貼り付け不要）。

---

以下の指示に従って、C# コードの構造化コードレビューを実施してください。

---

## 開始時の確認

まず以下を質問してください:

> レビュー対象のクラス名と、プロジェクト種別（ASP.NET Core Web API / MVC / WinForms / WPF / Console）を教えてください。

---

## C# レビューチェックリスト

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
  ```
- `[Authorize]` の漏れ → コントローラ / アクションへの認可なしアクセス → 全件確認
- `ModelState.IsValid` チェック漏れ → バリデーション未実施でビジネスロジックに進む
- CORS: `AllowAnyOrigin().AllowAnyMethod().AllowAnyHeader()` → 全開放。理由を確認

### D. EF Core の N+1 問題

- LAZY ローディングが有効で、ループ内に関連エンティティアクセス → N+1
  ```csharp
  // 危険
  var users = await _db.Users.ToListAsync();
  foreach (var user in users) { Console.WriteLine(user.Orders.Count); } // N+1

  // 対策
  var users = await _db.Users.Include(u => u.Orders).ToListAsync();
  ```
- `AsNoTracking()` の欠如 → 読み取り専用クエリでのトラッキングは不要

### E. 例外処理

- `catch (Exception ex) {}` での飲み込み → 記録・修正
- `throw ex;` ではなく `throw;` を使っているか（スタックトレース保持）
- グローバル例外ハンドラの欠如 → スタックトレースがクライアントに漏れる

### F. 設定・シークレット管理

- `appsettings.json` への直接的な接続文字列 / APIキー記載 → 環境変数 / Azure Key Vault 推奨
- `Development` 環境の設定が `Production` にも適用される構造 → 環境別設定ファイルの確認

---

## レビュー結果の出力形式

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

## モダナイゼーション判定

| 項目 | リスク | 検出数 |
|---|---|---|
| `async void` | 高 | N |
| 文字列補間 SQL（SQL Injection） | 高 | N |
| `HttpClient` の都度 `new` | 高 | N |
| EF Core N+1（LAZY 未解決） | 中 | N |
| `.Result` / `.Wait()` | 中 | N |
| IDisposable `using` 漏れ | 中 | N |
| `[Authorize]` 漏れ | 高 | N |

---

## 絶対禁止事項

- **`async void` の見落とし禁止** — イベントハンドラ以外の全 `async void` を記録
- **`ExecuteSqlRaw` の文字列連結・補間を見落とし禁止** — SQLインジェクション直結
- **「〜の可能性があります」禁止** — 断定する
- **途中停止禁止** / **AI voice禁止**
