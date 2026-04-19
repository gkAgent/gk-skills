# Spec Gen — C# × SQL Server

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-csharp-sqlserver` で自動起動します（貼り付け不要）。

---

以下の指示に従って、C# × SQL Server システムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。
> このスキルは C# (.NET) × SQL Server に特化しています。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> C# × SQL Server のプロジェクトを教えてください。
>
> - **.NET バージョン**: .NET 6 / 7 / 8 / 9 / Framework 4.x
> - **プロジェクト種別**: ASP.NET Core MVC / Web API / WinForms / WPF / コンソール
> - **データアクセス**: EF Core / Dapper / ADO.NET / ストアドプロシージャ
> - **SQL Server バージョン**: 2019 / 2022 / Azure SQL

---

## Phase 1: 質問フェーズ

**Stage 1 — 基本情報**
1. システム名と対象機能は？
2. `*.csproj` / `DbContext` クラス / `appsettings.json` は添付できますか？

**Stage 2 — C# .NET 固有情報**
- EF Core: Code First / Database First？`Migrations/` フォルダはある？
- Dapper: `QueryAsync<T>` / `ExecuteAsync` のパラメータ方式
- ADO.NET: `SqlCommand.Parameters.AddWithValue` vs `Add` の使い分け
- ストアドプロシージャ: `CommandType.StoredProcedure` 使用箇所

**Stage 3 — SQL Server 固有情報**
- トランザクション: `SqlTransaction` / `TransactionScope` / EF Core `BeginTransaction`
- 接続文字列の暗号化設定（`Encrypt=True`）
- 楽観ロック（`rowversion` / `timestamp`）の使用有無

---

## Phase 3: 仕様書生成 — C# × SQL Server テンプレート

```markdown
# 詳細仕様書 — [機能名]

## 0. 基本情報
| .NET | データアクセス | SQL Server |
|---|---|---|
| .NET 8 | EF Core 8 | SQL Server 2022 |

## 3. DB操作
| 種別 | テーブル/SP | 条件 | EF Core / ADO.NET |
|---|---|---|---|
| SELECT | Users | Id = @id | `FindAsync(id)` |
| INSERT | Orders | - | `AddAsync` + `SaveChangesAsync` |
| EXEC | sp_GetUser | @userId | `CommandType.StoredProcedure` |

## 4. トランザクション管理
| 方式 | 箇所 | ロールバック条件 |
|---|---|---|
| `SaveChangesAsync` | 一括保存 | 例外時に自動ロールバック |

## 5. IDisposable リソース管理
| リソース | using 文 | 注意事項 |
|---|---|---|
| `SqlConnection` | ✅ | `await using` 推奨 |

## 6. TODO / 要確認事項
```

---

## 絶対禁止事項

- **`using` なしのリソース使用禁止**: `SqlConnection`/`SqlCommand`がusingで囲まれていない箇所は「リソースリーク: 要修正」と記載
- **`AddWithValue` の型不一致警告**: 型が曖昧な場合は明示的な `SqlDbType` 指定を推奨として記載
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
