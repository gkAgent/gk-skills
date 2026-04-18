---
name: code-review-vbnet
description: Structured code review for VB.NET codebases (WinForms, WebForms, WPF, .NET Framework and .NET 6+). Use when reviewing legacy VB.NET code before production changes, assessing VB.NET code quality for modernization planning, reviewing VB.NET × SQL Server systems, or auditing VB.NET code inherited from a previous team. Extends code-review-enterprise-ja. Covers Option Strict Off risks, On Error GoTo patterns, IDisposable/SqlConnection leaks, late binding, Module global state, WebForms ViewState vulnerabilities, and SQL injection via string concatenation.
---

# Code Review — VB.NET

> Full coverage across 5 languages: install **code-review-enterprise-ja**.
> This skill focuses on VB.NET — the language that outlived everyone's predictions.

---

## Activation

Ask: レビュー対象のフォーム名/モジュール名と、プロジェクト種別（WinForms/WebForms/WPF）を教えてください。

---

## VB.NET Review Checklist

### A. Option Strict / 型安全（最重要）

- **`Option Strict Off`** が宣言されているか → Off なら以下を全件チェック
  - 暗黙的な数値変換（`Integer` → `Long` 等）
  - `Object` 型変数へのレイトバインディング: `obj.SomeMethod()` のような記述
  - `CType` / `DirectCast` / `TryCast` の使い分け（`DirectCast` は型不一致で例外）
- `IsNothing()` と `Is Nothing` の混在 → どちらかに統一
- `String.IsNullOrEmpty` の使用漏れ（空文字 + Nothing の両方チェックが必要な箇所）

### B. エラー処理

- **`On Error GoTo`** の使用 → `Try/Catch/Finally` への変換推奨箇所として全件記録
- `On Error Resume Next` の使用 → **危険**: エラーを無視して続行するパターン。即記録
- `Catch ex As Exception` での飲み込み（`ex` を使わずに続行）
- `Throw ex` ではなく `Throw` を使っているか（スタックトレース保持）

### C. リソース管理

- `SqlConnection` / `SqlCommand` / `SqlDataReader` が `Using` で囲まれているか
- `Dispose()` の明示的呼び出し漏れ
- `StreamReader` / `FileStream` の `Using` 漏れ

### D. セキュリティ

- **SQL インジェクション**: `"SELECT * FROM T WHERE ID = " & txtId.Text` 形式 → 即記録・即修正
- `SqlCommand.Parameters.Add` / `AddWithValue` の使用確認
- WebForms: `ViewState` の暗号化・MAC検証設定（`web.config` の `enableViewStateMac`）
- `Response.Redirect` の Open Redirect リスク（ユーザー入力をそのまま使用）
- XSS: `Response.Write` / `Label.Text` へのユーザー入力の直接代入

### E. アーキテクチャ / 品質

- `Module` 内の `Public` 変数（グローバル状態） → 全件一覧化
- `Form` 間の `Public` プロパティ経由のデータ受け渡し（結合度高）
- イベントハンドラへのビジネスロジック集中（100行超のイベントハンドラ）
- `GoTo` 文の使用 → 記録（スパゲッティコードの指標）
- `Static` ローカル変数の使用 → 状態管理の不透明さとして記録

### F. .NET Framework 固有（4.x）

- `WebClient` / `HttpWebRequest` の使用 → `HttpClient` への置き換え推奨
- `Thread` の直接使用 → `Task` / `Async/Await` への置き換え推奨
- `ArrayList` / `Hashtable` の使用 → ジェネリックコレクションへの置き換え

---

## Review Report Output

```markdown
## レビュー結果: [ファイル名]

### 重大（即修正）
- [行番号] SQL文字列連結によるSQLインジェクション脆弱性。Parameters.Addに変更せよ
- [行番号] On Error Resume Next — エラーを無視して続行。即刻除去せよ

### 警告（修正推奨）
- [行番号] SqlConnection が Using なし。リソースリークの可能性
- Option Strict Off により全ファイルで暗黙型変換が発生。Strict On への移行を推奨

### 情報
- On Error GoTo が [N]箇所。モダナイゼーション時の優先変換対象
- Module.GlobalVar が [N]個。設計上の結合度を文書化

### 総評
[断定口調で1〜3行]
```

---

## Modernization Assessment（モダナイゼーション判定）

レビュー後、以下のスコアを出力する:

| 項目 | リスク | 検出数 |
|---|---|---|
| `On Error GoTo` / `Resume Next` | 高 | N |
| SQL文字列連結 | 高 | N |
| `Option Strict Off` | 高 | - |
| `Using` なしリソース | 中 | N |
| グローバル変数（Module） | 中 | N |
| `GoTo` 文 | 中 | N |
| レイトバインディング | 低〜中 | N |

**VB6からの移行度**: [VB6的パターンが多い / 部分的にモダン / ほぼモダン]

---

## Forbidden

- **`On Error Resume Next` の見落とし禁止** — これが最も危険なパターン。必ず発見して報告
- **Option Strict Off の無視禁止** — 宣言状態を冒頭で確認し、Off なら全体に影響することを明記
- **「〜の可能性があります」禁止** — 断定する
- **途中停止禁止** / **AI voice禁止**
