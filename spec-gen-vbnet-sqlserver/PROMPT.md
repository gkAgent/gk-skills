# Spec Gen — VB.NET × SQL Server

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-vbnet-sqlserver` で自動起動します（貼り付け不要）。

---

以下の指示に従って、VB.NET × SQL Server システムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。
> このスキルは VB.NET × SQL Server — 日本レガシーエンタープライズシステムの最多組み合わせ — に特化しています。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> VB.NET × SQL Server のプロジェクトを教えてください。
>
> - **.NET バージョン**: .NET Framework 4.x / .NET 6+ / VB6からの移行途中
> - **プロジェクト種別**: WinForms / WebForms / WPF / クラスライブラリ
> - **データアクセス**: ADO.NET (SqlCommand) / DataAdapter+DataSet / ストアドプロシージャ / Entity Framework
> - **Option Strict**: On / Off（Off の場合は特に注意が必要）

---

## Phase 1: 質問フェーズ

**Stage 1 — 基本情報**
1. システム名・機能名・主要フォーム名は？
2. `.vbproj` / `App.config` / `Web.config` は添付できますか？
3. `Option Strict On` が宣言されていますか？

**Stage 2 — VB.NET 固有情報**
- グローバル変数: `Module` に定義されている変数一覧
- エラー処理方式: `Try/Catch` / `On Error GoTo` / 混在
- イベントハンドラ: `Handles` 句 vs `AddHandler` の使い分け
- `WithEvents` / `Implements` の使用

**Stage 3 — SQL Server 固有情報**
- 接続文字列: `App.config` の `connectionStrings` セクション
- トランザクション: `SqlTransaction` / `TransactionScope`
- ストアドプロシージャ呼び出し: `CommandType.StoredProcedure`
- `DataAdapter.Fill` のクエリ内容

---

## Phase 3: 仕様書生成 — VB.NET × SQL Server テンプレート

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

## VB.NET 固有の追加確認事項

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

## 絶対禁止事項

- **Option Strict Off の無視禁止** — 宣言状態を必ず確認し、Off なら冒頭に「型安全性リスク」として明記
- **On Error GoTo の無視禁止** — 存在を必ず記録（モダナイゼーション計画の基礎情報）
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
