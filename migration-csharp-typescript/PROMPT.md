# Migration — C# (.NET) → TypeScript

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/migration-csharp-typescript` で自動起動します（貼り付け不要）。

---

以下の指示に従って、C# (.NET) システムの TypeScript 移行を支援してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 移行対象を教えてください。
>
> **現行**: .NET バージョン / プロジェクト種別（ASP.NET Core MVC / Web API / WinForms / WPF）
> **移行先**: TypeScript + Next.js / React / NestJS / Express
> **DB**: SQL Server / PostgreSQL / MySQL（移行有無も）
> **規模**: コントローラ数・画面数・エンドポイント数

---

## Phase 1: アセスメント

### 1.1 C# コードの分類

| 分類 | 内容 | 難易度 |
|---|---|---|
| LINQ クエリ | `.Where().Select().OrderBy()` | 低（TS 配列メソッドに対応） |
| Entity Framework Core | DbContext / Migrations | 中（Prisma/Drizzle に移行） |
| ASP.NET Core MVC Controller | `[HttpGet]` / `[Authorize]` | 中 |
| 依存性注入（DI） | `IServiceCollection` / `AddScoped` | 中（NestJS DI or 手動 DI） |
| ストアドプロシージャ呼び出し | `SqlCommand` / EF `FromSqlRaw` | 中〜高 |
| SignalR | リアルタイム通信 | 高（Socket.IO / tRPC subscriptions） |
| WinForms / WPF | UI層 | 高（React への UI再設計が必要） |
| COM Interop / Office 自動化 | Excel/Word 操作 | 高（要代替手段検討） |
| Background Service | `IHostedService` | 中（Node.js cron / BullMQ） |

### 1.2 移行不可・要判断

- `System.Windows.*` UI コンポーネント（WinForms 固有）
- P/Invoke（ネイティブ DLL 呼び出し）
- COM Interop（Excel.Interop 等）
- `System.Drawing` → `sharp` / `canvas` に代替
- `System.Security.Cryptography` → Node.js `crypto` モジュールで代替可能

---

## Phase 1.5: Designer ファイル徹底分析（WinForms / WPF 案件必須）

**`.Designer.cs` / `.designer.cs` を全ファイル分析する。省略禁止。**

### コントロール抽出テーブル（フォームごとに作成）

```
## [FormName].Designer.cs — コントロール一覧

| コントロール名 | 型 | 主要プロパティ | バインドイベント |
|---|---|---|---|
| btnSave | Button | Text="保存", TabIndex=5, Anchor=Bottom,Right | Click→btnSave_Click |
| txtUserName | TextBox | MaxLength=50, TabIndex=1, Enabled=true | TextChanged→ValidateInput |
| dgvOrders | DataGridView | AutoSizeColumnsMode=Fill, ReadOnly=false, MultiSelect=false | CellClick→dgvOrders_CellClick |
| cmbStatus | ComboBox | DropDownStyle=DropDownList, TabIndex=3 | SelectedIndexChanged→FilterOrders |
| pnlMain | Panel | Dock=Fill | — |
```

### 画面レイアウト記述

```
## [FormName] レイアウト概要

- Formサイズ: [Width] x [Height] px
- レイアウト構造:
  TopPanel(ToolStrip) / MainPanel(SplitContainer: 左=TreeView, 右=DataGridView) / BottomPanel(StatusStrip)
- タブ順序: txtUserId(1) → txtUserName(2) → cmbDept(3) → btnSearch(4) → btnSave(5)
- 必須入力コントロール: [コントロール名リスト]
- 読み取り専用コントロール: [コントロール名リスト]
```

### 画面責務の1行定義

```
| フォーム名 | 責務（1行） | コントロール数 | 複雑度 |
|---|---|---|---|
| FrmOrderEntry | 受注入力 — 顧客選択・商品追加・数量入力・合計計算・登録 | 24 | 高 |
| FrmOrderList | 受注一覧 — 検索・絞り込み・CSV出力 | 12 | 中 |
| FrmMasterMaint | マスタ保守 — 追加/編集/削除/並び替え | 18 | 中 |
```

> **ASP.NET Core MVC / Web API 案件**: Designer ファイルは存在しない。
> 代わりに コントローラ + View (.cshtml) を同等分析する。
> コントローラ責務1行定義表（フォーム一覧と同じ構造）を作成すること。

---

## Phase 1.6: 分析結果 → 移行設計書

**Phase 1 + 1.5 の結果を「移行設計書」にまとめる。これが以降の全実装の唯一の根拠となる。**

```markdown
## 移行設計書 — [システム名]

### 1. 移行対象サマリ
- 画面/コントローラ数: N 件
- 移行困難項目: [COM Interop / SignalR / P/Invoke / etc.]
- DB: [テーブル数・EF Core モデル数・ストアドプロシージャ数]

### 2. 画面/コントローラごとの移行設計

#### [FrmOrderEntry / OrdersController] → OrderEntryPage.tsx

**元画面の構造（Phase 1.5 より）:**
- 顧客選択: cmbCustomer (ComboBox / SelectList, SelectedIndexChanged → 顧客情報自動入力)
- 明細グリッド: dgvItems (DataGridView / HTML Table, CellEndEdit → 金額再計算)
- 合計表示: lblTotal (Label / ViewData, ReadOnly)
- 登録ボタン: btnSave / [HttpPost] (バリデーション → DB INSERT)

**移行後の設計:**
- State: `{ customer, items[], total }` (useReducer推奨、複雑なため)
- API: `POST /api/orders` (バックエンド実装)
- バリデーション: 顧客必須・明細1件以上・数量>0
- EF Core Include クエリ → Prisma include に対応

**スプリント見積もり:** Front 2日 / Server 1日 / Test 1日 = 計4日

### 3. 移行順序（優先度順）
1. [LoginController / FrmLogin] — 認証基盤
2. [OrdersController / FrmOrderEntry] — 最重要業務画面
3. [OrdersController GET / FrmOrderList] — 参照系

### 4. 共通部品設計
- ErrorBoundary / Toast通知 / ローディング状態
- 認証コンテキスト (NextAuth / Clerk)
- DB接続 (Prisma Client singleton)
- LINQ クエリ → Prisma クエリ対応表
```

---

## Phase 2: 変換パターン集

### 2.1 型・変数

```csharp
// C#
int count = 0;
string name = "田中";
bool isActive = true;
List<string> items = new List<string>();
Dictionary<string, int> map = new();
```
```typescript
// TypeScript
let count: number = 0;
const name: string = "田中";
const isActive: boolean = true;
const items: string[] = [];
const map: Record<string, number> = {};
```

### 2.2 LINQ → 配列メソッド

```csharp
// C#
var result = users
    .Where(u => u.IsActive)
    .Select(u => new { u.Id, u.Name })
    .OrderBy(u => u.Name)
    .ToList();

var first = users.FirstOrDefault(u => u.Id == id);
var any = users.Any(u => u.Role == "admin");
var sum = orders.Sum(o => o.Amount);
```
```typescript
// TypeScript
const result = users
    .filter(u => u.isActive)
    .map(u => ({ id: u.id, name: u.name }))
    .sort((a, b) => a.name.localeCompare(b.name));

const first = users.find(u => u.id === id) ?? null;
const any = users.some(u => u.role === "admin");
const sum = orders.reduce((acc, o) => acc + o.amount, 0);
```

### 2.3 async/await

```csharp
// C#
public async Task<User?> GetUserAsync(int id)
{
    try {
        return await _db.Users.FindAsync(id);
    } catch (DbException ex) {
        _logger.LogError(ex, "DB error");
        throw;
    }
}
```
```typescript
// TypeScript
async function getUser(id: number): Promise<User | null> {
    try {
        return await prisma.user.findUnique({ where: { id } });
    } catch (error) {
        logger.error({ error }, "DB error");
        throw error;
    }
}
```

### 2.4 Entity Framework → Prisma

```csharp
// C# EF Core
public class User {
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public ICollection<Order> Orders { get; set; } = new List<Order>();
}
// Query
var users = await _db.Users
    .Include(u => u.Orders)
    .Where(u => u.IsActive)
    .ToListAsync();
```
```typescript
// Prisma schema
model User {
  id     Int     @id @default(autoincrement())
  name   String
  orders Order[]
}
// Query
const users = await prisma.user.findMany({
    where: { isActive: true },
    include: { orders: true },
});
```

### 2.5 ASP.NET Core Controller → Next.js API Route

```csharp
// C# ASP.NET Core
[ApiController, Route("api/[controller]")]
public class UsersController : ControllerBase {
    [HttpGet("{id}"), Authorize]
    public async Task<ActionResult<UserDto>> Get(int id) {
        var user = await _service.GetAsync(id);
        return user is null ? NotFound() : Ok(user);
    }
}
```
```typescript
// Next.js App Router (TypeScript)
// app/api/users/[id]/route.ts
export async function GET(req: Request, { params }: { params: { id: string } }) {
    const session = await getServerSession();
    if (!session) return Response.json({ error: "Unauthorized" }, { status: 401 });
    const user = await getUser(Number(params.id));
    if (!user) return Response.json({ error: "Not found" }, { status: 404 });
    return Response.json(user);
}
```

### 2.6 依存性注入

```csharp
// C# DI
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
```
```typescript
// NestJS DI（バックエンドAPIの場合）
@Injectable()
export class UserService { ... }

@Module({
    providers: [UserService, CacheService],
})
export class UserModule {}
```

---

## Phase 3: Sprint Planning — PDCAサイクル設計

**Phase 1.6 の移行設計書を唯一の入力とする。分析を活かさないまま実装に入ることを禁止する。**

### 3.0 事前共通タスク（Sprint 0 — 1回のみ）

```markdown
## Sprint 0: 環境構築（全画面共通）

- [ ] TypeScript + Next.js / NestJS プロジェクト初期化
- [ ] Prisma セットアップ + EF Core モデルから schema.prisma 変換
- [ ] 認証ライブラリ選定（NextAuth / Clerk）+ AuthContext 実装
- [ ] 共通コンポーネント雛形（ErrorBoundary / Toast / Loading）
- [ ] SQL Server → PostgreSQL マイグレーション判断（任意）
- [ ] CI/CD パイプライン（Vercel / GitHub Actions）
```

### 3.1 画面/コントローラ単位 PDCAサイクル（Sprint N ごとに繰り返す）

```markdown
## Sprint N: [FrmXxx / XxxController] → [XxxPage.tsx / /api/xxx]

### 前提（Phase 1.6 より転記）
- 元画面/コントローラの責務: [1行定義]
- 主要コントロール/アクション: [リスト]
- State設計: [useState / useReducer]
- API: [エンドポイント一覧]
- EF Core クエリ → Prisma 対応: [対応表]

### Step 1: 仕様書生成（Front設計）
- [ ] Phase 1.5 のコントロール一覧 → Props / State 定義に変換
- [ ] イベントハンドラ / Action → React ハンドラシグネチャ定義
- [ ] LINQ クエリ → TypeScript 配列メソッド対応表
- [ ] バリデーションルール → Zodスキーマ定義
- 成果物: `docs/specs/[XxxPage].spec.md`

### Step 2: フロント実装
- [ ] [XxxPage].tsx 作成（State + JSX + スタイル）
- [ ] フォームバリデーション（react-hook-form + Zod）
- [ ] ローディング / エラー表示
- [ ] ブラウザで元画面と目視比較（WinForms案件）
- 完了条件: 元画面のタブ順・必須入力・レイアウトが再現されている

### Step 3: サーバー実装
- [ ] `/api/[resource]` Route Handler / NestJS Controller 実装
- [ ] Prisma クエリ（EF Core `Include` → Prisma `include` 対応）
- [ ] `[Authorize]` → NextAuth session check / JWT middleware
- [ ] ストアドプロシージャの TypeScript 化（または保持判断）
- [ ] API 単体テスト（Jest + Supertest）

### Step 4: 結合テスト
- [ ] フロント ↔ サーバー E2E テスト（Playwright）
- [ ] 旧システムと同じ入力 → 同じ出力の確認
- [ ] SignalR 代替（Socket.IO）がある場合は接続テスト
- 完了条件: 旧システムとのデータ整合性が確認できている

### 振り返り（次 Sprint への申し送り）
- 想定外だったこと:
- LINQ → TS 変換で再利用できるパターン:
- Prisma スキーマに追記が必要な内容:
```

### 3.2 Sprint 計画一覧

```markdown
| Sprint | 画面/コントローラ | 複雑度 | Front | Server | Test | 合計 |
|--------|------------------|--------|-------|--------|------|------|
| Sprint 0 | 環境構築 | — | — | — | — | 1週 |
| Sprint 1 | [Login] | 低 | 1日 | 0.5日 | 0.5日 | 2日 |
| Sprint 2 | [XxxController] | 中 | 2日 | 1日 | 1日 | 4日 |
| Sprint N | [YyyForm] | 高 | 3日 | 2日 | 1日 | 6日 |
| — | 最終統合テスト | — | — | — | — | 3日 |

**見積もりルール**: 画面数・コントローラ数が判明した時点で更新する。今の数字は仮置き。
```

---

## Phase 4: リスク登録簿

| リスク | 影響 | 対策 |
|---|---|---|
| COM Interop (Excel等) | 移行不可 | サーバーサイドで ExcelJS / SheetJS に代替 |
| SignalR | 設計変更必要 | Socket.IO または Server-Sent Events |
| WinForms ビジネスロジック混入 | UI層との分離が必要 | サービス層を先に抽出してから UI 再設計 |
| 複雑なEF Core クエリ | Prisma で表現不可な場合あり | `$queryRaw` または Drizzle への切り替え |

---

## 絶対禁止事項

- **工数の根拠なし断言禁止** / **移行可否の独断禁止** / **AI voice禁止**
