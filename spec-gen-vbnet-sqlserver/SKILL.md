---
name: spec-gen-vbnet-sqlserver
description: Generates design specification documents (詳細仕様書) from VB.NET source code using SQL Server. Use when documenting legacy VB.NET WinForms, WebForms, or WPF applications backed by SQL Server, reverse-engineering undocumented VB.NET systems for modernization planning, or producing 詳細仕様書 for VB.NET × SQL Server enterprise systems. Extends spec-gen-enterprise-ja. Covers ADO.NET SqlCommand patterns, DataAdapter/DataSet, stored procedure calls, On Error GoTo error handling, Module/Shared state, and WebForms ViewState.
---

# Spec Gen — VB.NET × SQL Server

> Full capability: install **spec-gen-enterprise-ja** for all 5 languages × 4 DBs.
> This skill focuses on VB.NET × SQL Server — the most common combination in Japanese legacy enterprise systems.

---

## Activation

Ask the user:

> VB.NET × SQL Server のプロジェクトを教えてください。
>
> - **.NET バージョン**: .NET Framework 4.x / .NET 6+ / VB6からの移行途中
> - **プロジェクト種別**: WinForms / WebForms / WPF / クラスライブラリ
> - **データアクセス**: ADO.NET (SqlCommand) / DataAdapter+DataSet / ストアドプロシージャ / Entity Framework
> - **Option Strict**: On / Off（Off の場合は特に注意が必要）

---

## Phase 1: Question-Driven Setup

**Stage 1 — Basics**
1. システム名・機能名・主要フォーム名は？
2. `.vbproj` / `App.config` / `Web.config` は添付できますか？
3. `Option Strict On` が宣言されていますか？

**Stage 2 — VB.NET specifics**
- グローバル変数: `Module` に定義されている変数一覧
- エラー処理方式: `Try/Catch` / `On Error GoTo` / 混在
- イベントハンドラ: `Handles` 句 vs `AddHandler` の使い分け
- `WithEvents` / `Implements` の使用

**Stage 3 — SQL Server specifics**
- 接続文字列: `App.config` の `connectionStrings` セクション
- トランザクション: `SqlTransaction` / `TransactionScope`
- ストアドプロシージャ呼び出し: `CommandType.StoredProcedure`
- `DataAdapter.Fill` のクエリ内容

---

## Phase 3: Spec Output — VB.NET × SQL Server Template

```markdown
# 詳細仕様書 — [フォーム名/機能名]

## 0. 基本情報
| .NET | プロジェクト種別 | データアクセス | SQL Server |
|---|---|---|---|
| Framework 4.8 | WinForms | ADO.NET | 2019 |

## 2. 入力（フォームコントロール）
| コントロール名 | 種別 | 必須 | バリデーション |
|---|---|---|---|
| txtUserId | TextBox | ✅ | 数値チェック |

## 3. イベント処理
| イベント | ハンドラ | 処理概要 |
|---|---|---|
| btnSave.Click | btnSave_Click | 入力検証→DB保存 |

## 4. DB操作
| 種別 | テーブル/SP | 条件 | SqlCommand/SP名 |
|---|---|---|---|
| SELECT | T_USER | USER_ID = @userId | 直接SQL |
| EXEC | sp_SaveUser | @userId, @name | StoredProcedure |

## 5. Module/Shared グローバル変数
| 変数名 | 型 | 用途 | 変更タイミング |
|---|---|---|---|

## 6. エラー処理
| 方式 | 箇所 | ハンドリング内容 |
|---|---|---|
| Try/Catch | DB操作全般 | SqlException → メッセージボックス表示 |
| On Error GoTo | [旧来コード] | TODO: Try/Catch に変換推奨 |

## 7. TODO / 要確認事項
```

---

## VB.NET固有の追加確認事項

**Option Strict Off の場合（最重要）:**
- 暗黙的な型変換が多数存在する可能性 → 全て `TODO: 型変換の明示化` として記録
- レイトバインディング（`Object` 型への代入）→ 検出して全て列挙

**On Error GoTo の存在:**
- `On Error GoTo ErrorHandler` パターンが残っている箇所を全て列挙
- モダナイゼーション対象として記録（`Try/Catch` への変換必要）

**Module のグローバル状態:**
- `Module` 内の `Public` 変数は事実上のグローバル変数
- マルチフォーム間での共有状態として仕様書に明記

---

## Forbidden

- **Option Strict Off の無視禁止** — 宣言状態を必ず確認し、Off なら冒頭に「型安全性リスク」として明記
- **On Error GoTo の無視禁止** — 存在を必ず記録（モダナイゼーション計画の基礎情報）
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
