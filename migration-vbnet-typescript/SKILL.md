---
name: migration-vbnet-typescript
description: Guides VB.NET legacy system migration to TypeScript with a structured 5-phase methodology. Covers WinForms, WebForms, and WPF source patterns. Use when a user wants to modernize a VB.NET application to TypeScript/React or Next.js, assess migration feasibility, generate migration task lists, convert VB.NET code to TypeScript equivalents, or document a migration plan for stakeholder approval. Includes Oracle/SQL Server to PostgreSQL/MySQL migration guidance when applicable.
---

# Migration Kit — VB.NET to TypeScript

## Activation

When this skill is invoked, identify the migration scope and follow the 5-phase structure.

---

## Phase 0: Scope Identification

Ask the user:

> 移行対象を教えてください。
>
> **現行**: VB.NET バージョン / プロジェクト種別（WinForms / WebForms / WPF / コンソール）
> **移行先**: TypeScript + React / Next.js / Vue / NestJS（API）/ その他
> **DB**: Oracle / SQL Server / PostgreSQL / MySQL（移行有無も）
> **規模**: フォーム数・画面数・ファイル数（わかる範囲で）

---

## Phase 1: Assessment — 移行アセスメント

### 1.1 VB.NET コードの分類

Analyze the provided source code and categorize:

| 分類 | 内容 | 移行難易度 |
|---|---|---|
| UIロジック | Form_Load / Button_Click / DataGridView操作 | 中 |
| ビジネスロジック | 業務ルール・計算ロジック | 低〜中 |
| DB操作 | SqlCommand / DataAdapter / ストアドプロシージャ | 中〜高 |
| 外部連携 | WebService / COM / ActiveX / ファイルI/O | 高 |
| グローバル変数 | Module / Shared メンバー | 中 |
| Winフォーム固有 | MDI / PrintDocument / Crystal Reports | 高 |

### 1.2 移行不可・要判断項目

Flag these for stakeholder decision:

- `Microsoft.VisualBasic.*`ネームスペースの依存
- COM Interop（Excel.Interop / Word.Interop 等）
- ActiveX コントロール
- VB6 互換機能（`On Error Goto` / `GoTo` / `DoEvents`）
- Crystal Reports / DataReport

---

## Phase 2: Conversion Patterns — 変換パターン集

### 2.1 基本構文変換

**変数・型宣言**
```vbnet
' VB.NET
Dim count As Integer = 0
Dim name As String = "田中"
Dim items As List(Of String) = New List(Of String)()
```
```typescript
// TypeScript
let count: number = 0;
const name: string = "田中";
const items: string[] = [];
```

**Null 判定**
```vbnet
' VB.NET
If obj Is Nothing Then ...
If String.IsNullOrEmpty(str) Then ...
```
```typescript
// TypeScript
if (obj === null || obj === undefined) { ... }
if (!str) { ... }  // または str === "" || str == null
```

**例外処理**
```vbnet
' VB.NET
Try
    ' 処理
Catch ex As SqlException
    MessageBox.Show(ex.Message)
Finally
    conn.Close()
End Try
```
```typescript
// TypeScript
try {
    // 処理
} catch (error) {
    if (error instanceof Error) console.error(error.message);
} finally {
    await conn.close();
}
```

### 2.2 イベント処理

**Button_Click → onClick ハンドラ**
```vbnet
' VB.NET (WinForms)
Private Sub btnSave_Click(sender As Object, e As EventArgs) Handles btnSave.Click
    SaveRecord()
End Sub
```
```typescript
// React
const handleSave = async () => {
    await saveRecord();
};
<button onClick={handleSave}>保存</button>
```

### 2.3 DB アクセス

**ADO.NET → Prisma/Drizzle**
```vbnet
' VB.NET (ADO.NET)
Using conn As New SqlConnection(connStr)
    conn.Open()
    Dim cmd As New SqlCommand("SELECT * FROM Users WHERE Id = @id", conn)
    cmd.Parameters.AddWithValue("@id", userId)
    Dim reader = cmd.ExecuteReader()
    While reader.Read()
        Dim name = reader("Name").ToString()
    End While
End Using
```
```typescript
// TypeScript (Prisma)
const user = await prisma.user.findUnique({ where: { id: userId } });

// TypeScript (raw SQL with parameterized query)
const users = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
```

**ストアドプロシージャ呼び出し**
```vbnet
' VB.NET
Dim cmd As New SqlCommand("sp_GetUser", conn)
cmd.CommandType = CommandType.StoredProcedure
cmd.Parameters.AddWithValue("@userId", userId)
```
```typescript
// TypeScript (pg raw)
const result = await pool.query('CALL sp_get_user($1)', [userId]);
// または Prisma $queryRaw
const result = await prisma.$queryRaw`EXEC sp_GetUser ${userId}`;
```

### 2.4 画面状態管理

**Form のプロパティ → React State**
```vbnet
' VB.NET (WinForms)
Public Property CurrentUser As String
    Get
        Return txtUserName.Text
    End Get
    Set(value As String)
        txtUserName.Text = value
    End Set
End Property
```
```typescript
// React
const [currentUser, setCurrentUser] = useState<string>("");
// JSX
<input value={currentUser} onChange={(e) => setCurrentUser(e.target.value)} />
```

---

## Phase 3: Migration Task List — タスクリスト生成

Generate a structured migration task list:

```markdown
## 移行タスクリスト

### フェーズ A: 環境構築
- [ ] TypeScript + React/Next.js プロジェクト初期化
- [ ] DB クライアント (Prisma/Drizzle) セットアップ
- [ ] 認証ライブラリ選定・設定

### フェーズ B: データ層移行
- [ ] [テーブル名] のモデル定義 (Prismaスキーマ)
- [ ] ストアドプロシージャの純粋SQL化 or 保持判断
- [ ] DBマイグレーションスクリプト作成

### フェーズ C: ビジネスロジック移行
- [ ] [機能名] ロジックの TypeScript 関数化
- [ ] テスト（Jest）作成

### フェーズ D: UI移行
- [ ] [画面名] の React コンポーネント作成
- [ ] フォームバリデーション実装
- [ ] エラー表示

### フェーズ E: 結合・テスト
- [ ] E2E テスト（Playwright/Cypress）
- [ ] 旧システムとのデータ整合性確認
```

---

## Phase 4: Risk Register — リスク登録簿

For each high-difficulty item from Phase 1, create a risk entry:

| リスク | 影響 | 対策 |
|---|---|---|
| COM Interop 依存 | 移行不可の可能性 | サーバーサイド API で代替か、別途 Windows サービス維持 |
| `On Error GoTo` 多用 | エラーフロー不明瞭 | Try/Catch への変換時に業務ルール確認必須 |
| グローバル変数（Module） | 状態管理の複雑化 | React Context / Zustand / Redux への段階的移行 |

---

## Phase 5: Stakeholder Summary — ステークホルダー向け説明資料

Generate a non-technical summary:

```markdown
## 移行概要（経営・管理職向け）

### 何をするか
[現行システム名] を [移行先技術] に移行します。

### なぜ今か
- VB.NET / .NET Framework のサポート期限: [年月]
- 現状の維持コスト vs 移行コスト

### 移行期間の見積もり（概算）
| フェーズ | 内容 | 期間 |
|---|---|---|
| A: 環境構築 | 1週間 |
| B: データ層 | [機能数]機能 | X週間 |
| C: ロジック | [機能数]機能 | X週間 |
| D: UI | [画面数]画面 | X週間 |
| E: テスト | 2週間 |

### リスクと軽減策
[Phase 4 のリスク上位3件を記載]
```

---

## Forbidden — 絶対禁止事項

- **推測での工数見積もり禁止**: 「だいたい3ヶ月」などの根拠なし見積もりを出さない。「[画面数]画面、[機能数]機能が判明した時点で見積もり直す」と明記
- **移行可否の独断禁止**: COM Interop など高リスク項目は「要ステークホルダー判断」と明示
- **コード変換の動作保証禁止**: 変換パターンはパターン例であり、動作確認は利用者の責任と明記
- **AI voice 禁止**: 「モダンな技術スタックへの移行により〜」などの営業文句は書かない
