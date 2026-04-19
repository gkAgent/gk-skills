# Migration Kit — VB.NET → TypeScript

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/migration-vbnet-typescript` で自動起動します（貼り付け不要）。

---

以下の指示に従って、VB.NET システムの TypeScript 移行を支援してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 移行対象を教えてください。
>
> **現行**: VB.NET バージョン / プロジェクト種別（WinForms / WebForms / WPF / コンソール）
> **移行先**: TypeScript + React / Next.js / Vue / NestJS（API）/ その他
> **DB**: Oracle / SQL Server / PostgreSQL / MySQL（移行有無も）
> **規模**: フォーム数・画面数・ファイル数（わかる範囲で）

---

## Phase 1: 移行アセスメント

### 1.1 VB.NET コードの分類

| 分類 | 内容 | 移行難易度 |
|---|---|---|
| UIロジック | Form_Load / Button_Click / DataGridView操作 | 中 |
| ビジネスロジック | 業務ルール・計算ロジック | 低〜中 |
| DB操作 | SqlCommand / DataAdapter / ストアドプロシージャ | 中〜高 |
| 外部連携 | WebService / COM / ActiveX / ファイルI/O | 高 |
| グローバル変数 | Module / Shared メンバー | 中 |
| Winフォーム固有 | MDI / PrintDocument / Crystal Reports | 高 |

### 1.2 移行不可・要判断項目

以下はステークホルダー判断が必要な項目として明示してください:

- `Microsoft.VisualBasic.*`ネームスペースの依存
- COM Interop（Excel.Interop / Word.Interop 等）
- ActiveX コントロール
- VB6 互換機能（`On Error Goto` / `GoTo` / `DoEvents`）
- Crystal Reports / DataReport

---

## Phase 1.5: Designer ファイル徹底分析（WinForms / WPF 案件必須）

**`.designer.vb` / `.Designer.vb` を全ファイル分析する。省略禁止。**

### コントロール抽出テーブル（フォームごとに作成）

```
## [FormName].designer.vb — コントロール一覧

| コントロール名 | 型 | 主要プロパティ | バインドイベント |
|---|---|---|---|
| btnSave | Button | Text="保存", TabIndex=5, Anchor=Bottom,Right | Click→btnSave_Click |
| txtUserName | TextBox | MaxLength=50, TabIndex=1, Enabled=True | TextChanged→ValidateInput |
| dgvOrders | DataGridView | AutoSizeColumns=Fill, ReadOnly=False, MultiSelect=False | CellClick→dgvOrders_CellClick |
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

### 画面スクリーンショット取得（推奨）

元システムの画面イメージを以下のいずれかで保持する:

```
方法1（Visual Studio がある場合）:
  - .sln を開いてフォームをデザイナーで表示 → Ctrl+PrintScreen → ペイント等で保存
  - 保存先: docs/screens/[FormName].jpg

方法2（ソースのみの場合）:
  - designer.vb の Location/Size 情報から draw.io / Figma でワイヤーフレーム再現
  - 精度は落ちるが「元画面の構造」が視覚化できれば十分

方法3（Claude に描かせる）:
  - designer.vb を貼り付けて「このコントロール配置をASCIIアートで表現してください」
  - React への変換時の参照用として保持
```

### 画面責務の1行定義

```
| フォーム名 | 責務（1行） | コントロール数 | 複雑度 |
|---|---|---|---|
| FrmOrderEntry | 受注入力 — 顧客選択・商品追加・数量入力・合計計算・登録 | 24 | 高 |
| FrmOrderList | 受注一覧 — 検索・絞り込み・CSV出力 | 12 | 中 |
| FrmMasterMaint | マスタ保守 — 追加/編集/削除/並び替え | 18 | 中 |
```

**この表が後の PDCA スプリント設計の入力になる。**

---

## Phase 1.6: 分析結果 → 移行設計書

**Phase 1 + 1.5 の結果を「移行設計書」にまとめる。これが以降の全実装の唯一の根拠となる。**

```markdown
## 移行設計書 — [システム名]

### 1. 移行対象サマリ
- フォーム数: N 画面
- 移行困難項目: [COM Interop / On Error GoTo / etc.]
- DB: [テーブル数・ストアドプロシージャ数]

### 2. 画面ごとの移行設計

#### [FrmOrderEntry] → OrderEntryPage.tsx

**元画面の構造（Phase 1.5 より）:**
- 顧客選択: cmbCustomer (ComboBox, SelectedIndexChanged → 顧客情報を自動入力)
- 明細グリッド: dgvItems (DataGridView, CellEndEdit → 金額再計算)
- 合計表示: lblTotal (Label, ReadOnly)
- 登録ボタン: btnSave (Validation → DB INSERT)

**移行後の設計:**
- State: `{ customer, items[], total }` (useReducer推奨、複雑なため)
- API: `POST /api/orders` (バックエンド実装)
- バリデーション: 顧客必須・明細1件以上・数量>0
- 元の `dgvItems.CellEndEdit` → `onChange` で都度合計再計算

**スプリント見積もり:** Front 2日 / Server 1日 / Test 1日 = 計4日

### 3. 移行順序（優先度順）
1. [FrmLogin] — 認証基盤。他全画面の前提
2. [FrmOrderEntry] — 最重要業務画面
3. [FrmOrderList] — 参照系、依存少

### 4. 共通部品設計
- ErrorBoundary / Toast通知 / ローディング状態
- 認証コンテキスト (AuthContext)
- DB接続 (Prisma Client singleton)
```

---

## Phase 2: 変換パターン集

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

## Phase 3: Sprint Planning — PDCAサイクル設計

**Phase 1.6 の移行設計書を唯一の入力とする。分析を活かさないまま実装に入ることを禁止する。**

### 3.0 事前共通タスク（Sprint 0 — 1回のみ）

```markdown
## Sprint 0: 環境構築（全画面共通）

- [ ] TypeScript + React/Next.js プロジェクト初期化
- [ ] DB クライアント (Prisma/Drizzle) セットアップ・スキーマ定義
- [ ] 認証ライブラリ選定・設定（AuthContext 実装）
- [ ] 共通コンポーネント雛形（ErrorBoundary / Toast / Loading）
- [ ] ルーティング設計（画面一覧から逆算）
- [ ] CI/CD パイプライン（Vercel / GitHub Actions）
```

### 3.1 画面単位 PDCAサイクル（Sprint N ごとに繰り返す）

```markdown
## Sprint N: [FrmXxx] → [XxxPage.tsx]

### 前提（Phase 1.6 より転記）
- 元画面の責務: [1行定義]
- 主要コントロール: [リスト]
- State設計: [useStateまたはuseReducer]
- API: [エンドポイント一覧]
- バリデーション: [必須ルール]

### Step 1: 仕様書生成（Front設計）
- [ ] Phase 1.5 のコントロール一覧 → Props / State 定義に変換
- [ ] イベント一覧 → ハンドラ関数シグネチャ定義
- [ ] 元画面の Anchor/Dock 設定 → CSSレイアウト方針決定（Flex/Grid）
- [ ] バリデーションルール → Zodスキーマ定義
- 成果物: `docs/specs/[XxxPage].spec.md`

### Step 2: フロント実装
- [ ] [XxxPage].tsx 作成（State + JSX + スタイル）
- [ ] フォームバリデーション実装（react-hook-form + Zod）
- [ ] ローディング / エラー表示
- [ ] Storybook / ブラウザで元画面と目視比較
- 完了条件: 元画面のタブ順・必須入力・レイアウトが再現されている

### Step 3: サーバー実装
- [ ] `/api/[resource]` Route Handler 実装
- [ ] Prisma モデル / DBクエリ
- [ ] 元ストアドプロシージャの TypeScript 化（または保持判断）
- [ ] API 単体テスト（Jest + Supertest）

### Step 4: 結合テスト
- [ ] フロント ↔ サーバー E2E テスト（Playwright）
- [ ] 旧システムと同じ入力 → 同じ出力の確認
- [ ] エラーケース確認（必須未入力 / DB制約違反 / 通信エラー）
- 完了条件: 旧システムとのデータ整合性が確認できている

### 振り返り（次 Sprint への申し送り）
- 想定外だったこと:
- 次 Sprint に流用できるパターン:
- 設計書に追記が必要な共通部品:
```

### 3.2 Sprint 計画一覧

```markdown
| Sprint | 画面 | 複雑度 | Front | Server | Test | 合計 |
|--------|------|--------|-------|--------|------|------|
| Sprint 0 | 環境構築 | — | — | — | — | 1週 |
| Sprint 1 | [FrmLogin] | 低 | 1日 | 0.5日 | 0.5日 | 2日 |
| Sprint 2 | [FrmXxx] | 中 | 2日 | 1日 | 1日 | 4日 |
| Sprint N | [FrmYyy] | 高 | 3日 | 2日 | 1日 | 6日 |
| — | 最終統合テスト | — | — | — | — | 3日 |

**見積もりルール**: 画面数・複雑度が判明した時点で更新する。今の数字は仮置き。
```

---

## Phase 4: リスク登録簿

| リスク | 影響 | 対策 |
|---|---|---|
| COM Interop 依存 | 移行不可の可能性 | サーバーサイド API で代替か、別途 Windows サービス維持 |
| `On Error GoTo` 多用 | エラーフロー不明瞭 | Try/Catch への変換時に業務ルール確認必須 |
| グローバル変数（Module） | 状態管理の複雑化 | React Context / Zustand / Redux への段階的移行 |

---

## Phase 5: ステークホルダー向け説明資料

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
| A: 環境構築 | | 1週間 |
| B: データ層 | [機能数]機能 | X週間 |
| C: ロジック | [機能数]機能 | X週間 |
| D: UI | [画面数]画面 | X週間 |
| E: テスト | | 2週間 |

### リスクと軽減策
[Phase 4 のリスク上位3件を記載]
```

---

## 絶対禁止事項

- **推測での工数見積もり禁止**: 「だいたい3ヶ月」などの根拠なし見積もりを出さない。「[画面数]画面、[機能数]機能が判明した時点で見積もり直す」と明記
- **移行可否の独断禁止**: COM Interop など高リスク項目は「要ステークホルダー判断」と明示
- **コード変換の動作保証禁止**: 変換パターンはパターン例であり、動作確認は利用者の責任と明記
- **AI voice 禁止**: 「モダンな技術スタックへの移行により〜」などの営業文句は書かない
