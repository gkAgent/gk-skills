# Spec Generation — C# × PostgreSQL

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-csharp-postgresql` で自動起動します（貼り付け不要）。

---

以下の指示に従って、C# × PostgreSQL システムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

> モダン .NET クラウドネイティブの定番: ASP.NET Core × PostgreSQL。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 対象システムを教えてください。
>
> **C#**: .NET バージョン（6/7/8）/ フレームワーク（ASP.NET Core Web API / Minimal API / Worker Service）
> **ORM**: EF Core（Npgsql プロバイダ）/ Dapper / Npgsql 直接
> **PostgreSQL**: バージョン（14/15/16）/ 特殊機能（JSONB / 配列型 / UUID / 全文検索）
> **規模**: コントローラ数・エンティティ数・テーブル数

---

## PostgreSQL 固有パターン（C# 視点）

### JSONB カラム × EF Core

```csharp
// EF Core × Npgsql: JSONB マッピング
public class Order {
    public int Id { get; set; }

    [Column(TypeName = "jsonb")]
    public JsonDocument? Metadata { get; set; }  // System.Text.Json
}

// OnModelCreating での設定（Npgsql 8+）
builder.Entity<Order>()
    .Property(o => o.Metadata)
    .HasColumnType("jsonb");

// Dapper での JSONB 取得
using var conn = new NpgsqlConnection(connStr);
var result = await conn.QueryAsync<string>(
    "SELECT metadata::text FROM orders WHERE id = @id", new { id });
var json = JsonSerializer.Deserialize<Dictionary<string, object>>(result.First());
```

### 配列型（PostgreSQL Array）

```csharp
// EF Core × Npgsql: int[] → integer[] カラム
public class Tag {
    public int Id { get; set; }
    public int[] CategoryIds { get; set; } = Array.Empty<int>();  // integer[]
    public string[] Labels { get; set; } = Array.Empty<string>();  // text[]
}

// PostgreSQL 配列演算子: @> (contains), && (overlap)
var results = await _db.Tags
    .Where(t => t.CategoryIds.Contains(categoryId))  // Npgsql が @> に変換
    .ToListAsync();
```

### UUID 主キー

```csharp
// EF Core × PostgreSQL: UUID 主キー
public class Employee {
    public Guid Id { get; set; }  // UUID型
}

// OnModelCreating
builder.Entity<Employee>()
    .Property(e => e.Id)
    .HasDefaultValueSql("gen_random_uuid()");  // PostgreSQL 14+
```

### RETURNING 句（Dapper での挿入後取得）

```csharp
// Dapper: RETURNING で挿入後の全行を取得
var newEmployee = await conn.QuerySingleAsync<Employee>(
    @"INSERT INTO employees (name, dept_id)
      VALUES (@Name, @DeptId)
      RETURNING *",
    new { Name = name, DeptId = deptId });
```

### EF Core マイグレーション × Npgsql

```csharp
// Program.cs: Npgsql プロバイダ設定
builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseNpgsql(
        connectionString,
        npgsql => npgsql.MigrationsHistoryTable("__EFMigrationsHistory", "public")
    ));

// タイムゾーン設定（Npgsql v6+ では必須）
AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", false);
// DateTime → timestamptz には DateTimeKind.Utc を使用
```

---

## C# × PostgreSQL 仕様書テンプレート

```markdown
## [クラス名] — 詳細仕様

### クラス概要
| 項目 | 内容 |
|---|---|
| パッケージ | MyApp.Infrastructure.Repositories |
| 役割 | [業務機能の説明] |
| フレームワーク | ASP.NET Core 8 / EF Core 8 (Npgsql) |
| PostgreSQL バージョン | 16 |

### エンティティ定義（抜粋）
| プロパティ | C# 型 | PostgreSQL 型 | 備考 |
|---|---|---|---|
| Id | Guid | uuid | PK / gen_random_uuid() |
| Metadata | JsonDocument | jsonb | 構造: {key: string, value: ...} |
| CategoryIds | int[] | integer[] | カテゴリIDの配列 |
| CreatedAt | DateTime (UTC) | timestamptz | |

### Repository メソッド一覧
| メソッド名 | ORM | クエリ種別 | 説明 |
|---|---|---|---|
| FindByIdAsync | EF Core | findFirst | UUID で単件取得 |
| UpsertAsync | Dapper | INSERT … RETURNING | 挿入後のIDを返却 |
```

---

## 絶対禁止事項

- **`DateTime` の Kind 未設定禁止** — Npgsql v6+ は `DateTimeKind.Utc` 必須。`Unspecified` は例外になる場合あり
- **JSONB カラムの内部構造無記載禁止** — `jsonb` 型カラムは論理スキーマを仕様書に記載
- **配列型カラムの要素型・意味を無記載禁止** — `integer[]` の各要素が何を表すかを明記
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
