---
name: migration-csharp-typescript
description: Guides C# .NET migration to TypeScript with a structured 5-phase methodology. Covers ASP.NET Core MVC/Web API, WinForms, WPF, and class library patterns. Use when modernizing C# applications to TypeScript/React or Next.js, assessing C# migration feasibility, converting C# LINQ to TypeScript equivalents, migrating Entity Framework Core models to Prisma/Drizzle, or documenting a migration plan for stakeholder approval. Includes SQL Server to PostgreSQL migration guidance.
---

# Migration — C# (.NET) → TypeScript

---

## Phase 0: Scope

Ask the user:

> 移行対象を教えてください。
>
> **現行**: .NET バージョン / プロジェクト種別（ASP.NET Core MVC / Web API / WinForms / WPF）
> **移行先**: TypeScript + Next.js / React / NestJS / Express
> **DB**: SQL Server / PostgreSQL / MySQL（移行有無も）
> **規模**: コントローラ数・画面数・エンドポイント数

---

## Phase 1: Assessment

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

## Phase 2: Conversion Patterns

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

## Phase 3: Migration Task List

```markdown
### フェーズ A: 環境構築
- [ ] TypeScript + Next.js / NestJS プロジェクト初期化
- [ ] Prisma セットアップ + schema.prisma 定義
- [ ] 認証ライブラリ選定（NextAuth / Clerk）

### フェーズ B: データ層
- [ ] EF Core モデル → Prisma schema 変換
- [ ] SQL Server → PostgreSQL マイグレーション（任意）
- [ ] ストアドプロシージャの純粋SQL化 or 保持判断

### フェーズ C: API 層
- [ ] [コントローラ名] → Next.js API Route / NestJS Controller 変換
- [ ] 認可ロジック（[Authorize] → middleware）

### フェーズ D: UI 層（WinForms/WPF の場合）
- [ ] [フォーム名] → React コンポーネント設計
- [ ] フォームバリデーション（react-hook-form / zod）

### フェーズ E: テスト・結合
- [ ] Jest 単体テスト
- [ ] Playwright E2E
```

---

## Phase 4: Risk Register

| リスク | 影響 | 対策 |
|---|---|---|
| COM Interop (Excel等) | 移行不可 | サーバーサイドで ExcelJS / SheetJS に代替 |
| SignalR | 設計変更必要 | Socket.IO または Server-Sent Events |
| WinForms ビジネスロジック混入 | UI層との分離が必要 | サービス層を先に抽出してから UI 再設計 |
| 複雑なEF Core クエリ | Prisma で表現不可な場合あり | `$queryRaw` または Drizzle への切り替え |

---

## Forbidden

- **工数の根拠なし断言禁止** / **移行可否の独断禁止** / **AI voice禁止**
