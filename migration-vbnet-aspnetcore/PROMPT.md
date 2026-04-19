# Migration — VB.NET → ASP.NET Core

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/migration-vbnet-aspnetcore` で自動起動します（貼り付け不要）。

---

以下の指示に従って、VB.NET システムの ASP.NET Core 移行を支援してください。フェーズを順番に進め、スキップしないでください。

> .NET エコシステム内での移行。フルスタックリライトより低リスク。
> VB.NET → TypeScript の場合は **migration-vbnet-typescript** を使うこと。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 移行対象を教えてください。
>
> **現行**: VB.NET バージョン / .NET Framework バージョン / プロジェクト種別
> **移行先**: ASP.NET Core MVC / Web API / Blazor Server / Blazor WASM / Razor Pages
> **言語**: VB.NET のまま / C# に変換する（どちらも ASP.NET Core で可能）
> **DB**: SQL Server（現行） → SQL Server 継続 / PostgreSQL 移行？

---

## Phase 1: アセスメント

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

## Phase 1.5: Designer / .aspx ファイル徹底分析（WinForms / WebForms 案件必須）

**`.designer.vb` / `.Designer.vb`（WinForms）または `.aspx` + コードビハインド `.aspx.vb`（WebForms）を全ファイル分析する。省略禁止。**

### コントロール抽出テーブル（画面ごとに作成）

```
## [FormName / PageName].designer.vb — コントロール一覧

| コントロール名 | 型 | 主要プロパティ | バインドイベント |
|---|---|---|---|
| btnSave | Button | Text="保存", TabIndex=5 | Click→btnSave_Click |
| txtUserName | TextBox | MaxLength=50, TabIndex=1 | TextChanged→ValidateInput |
| GridView1 | GridView（WebForms） | AutoGenerateColumns=False | RowCommand→GridView1_RowCommand |
| DropDownList1 | DropDownList | AutoPostBack=True | SelectedIndexChanged→FilterOrders |
```

### 画面責務の1行定義

```
| 画面名 | 責務（1行） | コントロール数 | 複雑度 | 移行先 |
|---|---|---|---|---|
| FrmOrderEntry | 受注入力 — 顧客選択・商品追加・登録 | 24 | 高 | Razor Pages / Blazor |
| OrderList.aspx | 受注一覧 — 検索・絞り込み・CSV出力 | 12 | 中 | Razor Pages |
| MasterMaint.aspx | マスタ保守 — 追加/編集/削除 | 18 | 中 | Razor Pages |
```

**この表が後のスプリント設計の入力になる。**

---

## Phase 1.6: 分析結果 → 移行設計書

**Phase 1 + 1.5 の結果を「移行設計書」にまとめる。これが以降の全実装の唯一の根拠となる。**

```markdown
## 移行設計書 — [システム名]

### 1. 移行対象サマリ
- 画面数: N 画面（WinForms: X / WebForms: Y）
- 移行困難項目: [WebForms イベントモデル / Module グローバル / DataSet複雑バインド等]
- DB: [テーブル数・DataAdapter数・ストアドプロシージャ数]
- 移行先UI選択: Razor Pages / Blazor Server（理由: チームスキル・スコープ）

### 2. 画面ごとの移行設計

#### [OrderList.aspx] → OrderList.cshtml (Razor Pages)

**元画面の構造（Phase 1.5 より）:**
- 検索フォーム: txtKeyword + btnSearch (PostBack → GridView再バインド)
- 一覧: GridView1 (RowCommand → 編集/削除)
- ページング: GridView.PageIndexChanging

**移行後の設計:**
- PageModel: `OnGetAsync(string? keyword)` で検索
- ページング: `?page=N` クエリパラメータ + EF Core `.Skip().Take()`
- 削除: `OnPostDeleteAsync(int id)` + PRG パターン

**スプリント見積もり:** PageModel 1日 / EF Coreクエリ 0.5日 / テスト 0.5日 = 計2日

### 3. 移行順序（優先度順）
1. [Login.aspx / FrmLogin] — 認証基盤
2. [最重要業務画面] — 業務価値最大
3. 参照系画面 — 依存少

### 4. 共通部品設計
- TagHelper / ViewComponent 共通化
- EF Core DbContext + Migration
- DI サービス登録 (Program.cs)
- `On Error GoTo` → `try/catch` 機械的変換完了後に移行開始
```

---

## Phase 2: 変換パターン集

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

## Phase 3: Sprint Planning — PDCAサイクル設計

**Phase 1.6 の移行設計書を唯一の入力とする。分析を活かさないまま実装に入ることを禁止する。**

### 3.0 事前共通タスク（Sprint 0 — 1回のみ）

```markdown
## Sprint 0: 環境・基盤構築

- [ ] ASP.NET Core プロジェクト作成（.NET 8）
- [ ] EF Core セットアップ + 既存DB scaffold（テーブル→モデル自動生成）
- [ ] DI コンテナへ共通サービス登録（Program.cs）
- [ ] 認証: Forms Authentication → Cookie認証 / ASP.NET Core Identity
- [ ] `On Error GoTo` の全画面機械的変換（try/catch化）← 移行前の前提作業
- [ ] ロギング設定（Serilog / Microsoft.Extensions.Logging）
```

### 3.1 画面単位 PDCAサイクル（Sprint N ごとに繰り返す）

```markdown
## Sprint N: [.aspx / FrmXxx] → [XxxModel.cshtml.cs / XxxPage.razor]

### 前提（Phase 1.6 より転記）
- 元画面の責務: [1行定義]
- 主要コントロール: [GridView / DropDownList / TextBox リスト]
- PostBackイベント → PageModel メソッド対応表
- EF Core クエリ設計: [DataAdapter/DataSet の置き換え]

### Step 1: 仕様書生成（PageModel 設計）
- [ ] Phase 1.5 のコントロール一覧 → BindProperty / PageModel プロパティ定義
- [ ] PostBackイベント → OnGet / OnPost / OnPostXxx メソッド定義
- [ ] GridView RowCommand → BindProperty + OnPostDeleteAsync / OnPostEditAsync
- [ ] バリデーション: DataAnnotations / FluentValidation 定義
- 成果物: `docs/specs/[XxxPage].spec.md`

### Step 2: PageModel / Blazor コンポーネント実装
- [ ] `XxxModel.cshtml.cs` PageModel 実装
- [ ] `.cshtml` Razor マークアップ（サーバーコントロール → Tag Helpers）
- [ ] ブラウザで元画面と目視比較
- 完了条件: PostBack相当の動作が Razor PageModel で再現されている

### Step 3: データ層実装
- [ ] EF Core クエリ実装（DataAdapter → DbContext クエリ）
- [ ] ストアドプロシージャ: `FromSqlRaw` / `ExecuteSqlRawAsync` または PureSQL化
- [ ] xUnit 単体テスト（サービス層）

### Step 4: 結合テスト
- [ ] Playwright / Selenium E2E テスト
- [ ] 旧システムと同じ操作 → 同じデータ結果の確認
- 完了条件: 旧システムとのデータ整合性が確認できている

### 振り返り（次 Sprint への申し送り）
- DataSet バインディングで詰まった箇所:
- EF Core Include で追加が必要なリレーション:
- 次 Sprint に流用できる TagHelper パターン:
```

### 3.2 Sprint 計画一覧

```markdown
| Sprint | 画面 | 複雑度 | PageModel | EF Core | Test | 合計 |
|--------|------|--------|-----------|---------|------|------|
| Sprint 0 | 環境構築 | — | — | — | — | 1週 |
| Sprint 1 | [Login] | 低 | 0.5日 | 0.5日 | 0.5日 | 1.5日 |
| Sprint 2 | [一覧系画面] | 中 | 1日 | 1日 | 1日 | 3日 |
| Sprint N | [入力系画面] | 高 | 2日 | 1日 | 1日 | 4日 |
| — | 最終統合テスト | — | — | — | — | 2日 |

**見積もりルール**: 画面数が判明した時点で更新する。今の数字は仮置き。
```

---

## Phase 4: リスク登録簿

| リスク | 影響 | 対策 |
|---|---|---|
| WebForms のイベントモデル依存 | PostBackロジックの全面再設計 | ビジネスロジックを先にサービス層に抽出 |
| `Module` グローバル変数の多用 | DI 移行に時間がかかる | まず HttpContext/Session に退避、段階的にDI化 |
| 複雑な DataSet バインディング | GridView 等の完全廃止 | EF Core + HTML Table / Blazor Grid に置き換え |
| VB.NET のままASP.NET Coreに移行 | 将来的に C# への移行コストが残る | 段階1: VB.NET → ASP.NET Core、段階2: C# 化の2フェーズで計画 |

---

## 絶対禁止事項

- **工数の根拠なし断言禁止** / **AI voice禁止** / **途中停止禁止**
