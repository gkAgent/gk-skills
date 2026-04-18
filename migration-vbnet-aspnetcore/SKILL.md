---
name: migration-vbnet-aspnetcore
description: Guides VB.NET legacy system migration to ASP.NET Core (same Microsoft ecosystem, lower risk than full-stack rewrite). Covers VB.NET WinForms, WebForms, and class library migration to ASP.NET Core MVC/Web API, optionally with Blazor or React frontend. Use when modernizing VB.NET to stay within the .NET ecosystem, when a TypeScript rewrite is too large in scope, or when the team has existing C#/.NET expertise. Includes WebForms → Razor Pages migration, DataSet → EF Core, and On Error GoTo → structured exception handling.
---

# Migration — VB.NET → ASP.NET Core

> Staying within .NET. Lower risk than a full-stack rewrite.
> For VB.NET → TypeScript, see **migration-vbnet-typescript**.

---

## Phase 0: Scope

Ask the user:

> 移行対象を教えてください。
>
> **現行**: VB.NET バージョン / .NET Framework バージョン / プロジェクト種別
> **移行先**: ASP.NET Core MVC / Web API / Blazor Server / Blazor WASM / Razor Pages
> **言語**: VB.NET のまま / C# に変換する（どちらも ASP.NET Core で可能）
> **DB**: SQL Server（現行） → SQL Server 継続 / PostgreSQL 移行？

---

## Phase 1: Assessment

### 1.1 なぜ ASP.NET Core か（TypeScript でなく）

| 判断基準 | ASP.NET Core が有利 | TypeScript が有利 |
|---|---|---|
| チームスキル | .NET 経験者が多い | フロントエンド経験者が多い |
| スコープ | 既存ロジックの再利用が主 | UI の全面刷新が主 |
| リスク許容度 | 段階的移行（.NET同士） | 大規模リライト許容 |
| フロントエンド要件 | 管理画面・B2B | SPA・消費者向け |

### 1.2 移行パス分類

| 現行 | 移行先 | 難易度 |
|---|---|---|
| WebForms (.aspx) | Razor Pages / MVC View | 中（イベントモデルの廃止） |
| WinForms | Blazor Server | 中〜高（UI再設計） |
| VB.NET クラスライブラリ | .NET 6+ クラスライブラリ（VB.NET継続） | 低 |
| ADO.NET DataSet | Entity Framework Core | 中 |
| `On Error GoTo` | `try/catch` | 低（機械的変換） |
| `Module` グローバル変数 | DI コンテナ (Scoped/Singleton) | 中 |
| `Response.Redirect` / PostBack | Razor Pages PageModel | 中 |

---

## Phase 2: Conversion Patterns

### 2.1 エラー処理

```vbnet
' VB.NET (On Error GoTo)
Private Sub SaveUser()
    On Error GoTo ErrorHandler
    Dim conn As New SqlConnection(connStr)
    conn.Open()
    ' ...処理...
    Exit Sub
ErrorHandler:
    MessageBox.Show(Err.Description)
End Sub
```
```vbnet
' VB.NET (Try/Catch — ASP.NET Core対応)
Private Async Function SaveUserAsync() As Task
    Try
        Await Using conn = New SqlConnection(connStr)
            Await conn.OpenAsync()
            ' ...処理...
        End Using
    Catch ex As SqlException
        _logger.LogError(ex, "DB保存エラー")
        Throw
    End Try
End Function
```

### 2.2 WebForms → Razor Pages

```aspx
<%-- WebForms (.aspx) --%>
<asp:TextBox ID="txtName" runat="server" />
<asp:Button ID="btnSave" runat="server" Text="保存"
            OnClick="btnSave_Click" />
```
```csharp
// Razor Pages (.cshtml.cs PageModel)
public class EditModel : PageModel {
    [BindProperty]
    public string Name { get; set; } = "";

    public async Task<IActionResult> OnPostAsync() {
        if (!ModelState.IsValid) return Page();
        await _service.SaveAsync(Name);
        return RedirectToPage("./Index");
    }
}
```

### 2.3 DataSet → Entity Framework Core

```vbnet
' VB.NET (DataAdapter + DataSet)
Dim da As New SqlDataAdapter("SELECT * FROM Users WHERE IsActive=1", conn)
Dim ds As New DataSet()
da.Fill(ds, "Users")
For Each row As DataRow In ds.Tables("Users").Rows
    Dim name = row("UserName").ToString()
Next
```
```csharp
// EF Core
var users = await _db.Users
    .Where(u => u.IsActive)
    .ToListAsync();
foreach (var user in users) {
    var name = user.UserName;
}
```

### 2.4 依存性注入（Module → DI）

```vbnet
' VB.NET Module (グローバル状態)
Module AppState
    Public CurrentUserId As Integer
    Public ConnectionString As String = ConfigurationManager.ConnectionStrings("DB").ConnectionString
End Module
```
```csharp
// ASP.NET Core DI
// Program.cs
builder.Services.AddScoped<ICurrentUserService, CurrentUserService>();
builder.Services.AddSingleton<IConnectionStringProvider, ConnectionStringProvider>();

// 利用側
public class UserController(ICurrentUserService currentUser) : Controller { }
```

### 2.5 認証 (Forms Authentication → ASP.NET Core Cookie)

```xml
<!-- web.config WebForms -->
<authentication mode="Forms">
    <forms loginUrl="~/Login.aspx" timeout="30" />
</authentication>
```
```csharp
// ASP.NET Core Program.cs
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options => {
        options.LoginPath = "/Account/Login";
        options.ExpireTimeSpan = TimeSpan.FromMinutes(30);
    });
```

---

## Phase 3: Migration Task List

```markdown
### フェーズ A: 環境・基盤
- [ ] ASP.NET Core プロジェクト作成（.NET 8）
- [ ] EF Core セットアップ + 既存DBからscaffold
- [ ] DI コンテナへ共通サービス登録
- [ ] ロギング設定（Serilog / Microsoft.Extensions.Logging）

### フェーズ B: データ・ビジネスロジック
- [ ] DataSet/DataAdapter → EF Core クエリ変換
- [ ] ストアドプロシージャ: EF Core `FromSqlRaw` or `ExecuteSqlRawAsync`
- [ ] `On Error GoTo` → `try/catch` 機械的変換

### フェーズ C: Web層
- [ ] [.aspx画面名] → Razor Pages PageModel
- [ ] WebForms イベントハンドラ → PageModel メソッド
- [ ] 認証: Forms Auth → Cookie認証 / Identity

### フェーズ D: UI
- [ ] .aspx マークアップ → Razor (.cshtml) 変換
- [ ] サーバーコントロール → Tag Helpers / Razor コンポーネント

### フェーズ E: テスト
- [ ] xUnit 単体テスト
- [ ] Playwright / Selenium E2E
```

---

## Phase 4: Risk Register

| リスク | 影響 | 対策 |
|---|---|---|
| WebForms のイベントモデル依存 | PostBackロジックの全面再設計 | ビジネスロジックを先にサービス層に抽出 |
| `Module` グローバル変数の多用 | DI 移行に時間がかかる | まず HttpContext/Session に退避、段階的にDI化 |
| 複雑な DataSet バインディング | GridView 等の完全廃止 | EF Core + HTML Table / Blazor Grid に置き換え |
| VB.NET のままASP.NET Coreに移行 | 将来的に C# への移行コストが残る | 段階1: VB.NET → ASP.NET Core、段階2: C# 化の2フェーズで計画 |

---

## Forbidden

- **工数の根拠なし断言禁止** / **AI voice禁止** / **途中停止禁止**
