---
name: refactor-enterprise-ja
description: Guides refactoring of Japanese enterprise legacy code without full migration — improving code quality while staying in the same language and framework. Use when modernizing VB.NET/Java/C# code without switching languages, extracting service layers from fat WinForms/WebForms event handlers, replacing On Error GoTo with try/catch, decoupling SQL from UI code, or reducing Module-level global state. Covers all 5 enterprise languages (Java/TypeScript/C#/VB.NET/COBOL). Produces a refactoring plan with before/after code patterns and risk assessment.
---

# Refactor — Enterprise Legacy (Japanese SI)

> 移行せず、現行言語のまま品質を上げる。
> フルリライトより低リスク。移行前の前処理としても有効。

---

## Phase 0: Scope

Ask the user:

> リファクタ対象を教えてください。
>
> **言語**: Java / TypeScript / C# / VB.NET / COBOL
> **問題の症状**: イベントハンドラに全処理集中 / SQL がUI層に混在 / グローバル変数だらけ / エラー処理がない
> **目標**: テスト可能にする / 保守しやすくする / 移行の前処理として整理する
> **制約**: 言語・FW は変えない / デプロイ環境は変えない

---

## リファクタパターン一覧

### 1. Fat イベントハンドラ → サービス層抽出（VB.NET / C#）

```vbnet
' BEFORE: 1つのボタンクリックに300行のビジネスロジック
Private Sub btnSave_Click(sender As Object, e As EventArgs) Handles btnSave.Click
    ' バリデーション
    If txtEmpId.Text = "" Then MessageBox.Show("社員IDを入力してください") : Return
    ' DB接続
    Dim conn As New SqlConnection(ConfigurationManager.ConnectionStrings("DB").ConnectionString)
    conn.Open()
    ' SQL 直接実行
    Dim cmd As New SqlCommand("INSERT INTO EMPLOYEE ...", conn)
    ' ... 150行続く
End Sub
```

```vbnet
' AFTER: UI はサービス呼び出しのみ
Private Sub btnSave_Click(sender As Object, e As EventArgs) Handles btnSave.Click
    Dim request = New SaveEmployeeRequest(txtEmpId.Text, txtName.Text)
    Dim result = _employeeService.Save(request)
    If result.IsSuccess Then
        MessageBox.Show("保存しました")
    Else
        MessageBox.Show(result.ErrorMessage)
    End If
End Sub

' 抽出したサービス層（テスト可能）
Public Class EmployeeService
    Private ReadOnly _repository As IEmployeeRepository

    Public Function Save(request As SaveEmployeeRequest) As SaveResult
        If String.IsNullOrEmpty(request.EmpId) Then
            Return SaveResult.Fail("社員IDを入力してください")
        End If
        _repository.Insert(request)
        Return SaveResult.Success()
    End Function
End Class
```

### 2. On Error GoTo → Try/Catch（VB.NET）

```vbnet
' BEFORE: On Error GoTo
Private Sub SaveData()
    On Error GoTo ErrorHandler
    ' ... 処理 ...
    Exit Sub
ErrorHandler:
    LogError(Err.Description)
End Sub

' AFTER: Try/Catch
Private Sub SaveData()
    Try
        ' ... 処理 ...
    Catch ex As SqlException
        LogError($"DB エラー: {ex.Message}")
        Throw
    Catch ex As Exception
        LogError($"予期しないエラー: {ex.Message}")
        Throw
    End Try
End Sub
```

### 3. SQL の UI 層からの分離（全言語共通）

```java
// BEFORE: JSP/Servlet に SQL が直書き
public void doPost(HttpServletRequest req, HttpServletResponse resp) {
    String empId = req.getParameter("empId");
    Connection conn = DriverManager.getConnection(DB_URL);
    PreparedStatement ps = conn.prepareStatement(
        "SELECT * FROM EMPLOYEE WHERE EMP_ID = ?");
    ps.setString(1, empId);
    // ... 結果処理 ...
}

// AFTER: Repository パターンで分離
// Controller（UI 層）
@PostMapping("/employee")
public String show(@RequestParam String empId, Model model) {
    model.addAttribute("employee", employeeRepository.findById(empId));
    return "employee/detail";
}

// Repository（DB 層）
public interface EmployeeRepository {
    Optional<Employee> findById(String empId);
}
```

### 4. Module グローバル変数の解消（VB.NET）

```vbnet
' BEFORE: Module に全体共有状態
Module AppState
    Public CurrentUserId As Integer
    Public CurrentUserName As String
    Public DbConnectionString As String
End Module

' AFTER: クラス + DI（段階的に）
' Step 1: Module を Class に変換（まず動く状態にする）
Public Class UserContext
    Public Property UserId As Integer
    Public Property UserName As String

    Private Shared _instance As UserContext
    Public Shared ReadOnly Property Current As UserContext
        Get
            If _instance Is Nothing Then _instance = New UserContext()
            Return _instance
        End Get
    End Property
End Class
' Step 2: 依存を注入する形に段階的に移行
```

### 5. 定数マジックナンバーの除去

```java
// BEFORE
if (status.equals("A")) { ... }
if (rank.equals("1")) { ... }

// AFTER: enum / 定数クラス
public enum EmployeeStatus { ACTIVE("A"), INACTIVE("I"); }
public enum MemberRank { GOLD("1"), SILVER("2"), STANDARD("3"); }

if (status == EmployeeStatus.ACTIVE) { ... }
```

### 6. テスト不可能な static メソッドの解消

```java
// BEFORE: static メソッドで全処理（モック不可）
public class EmailUtil {
    public static void sendWelcome(String email) {
        // 実際にメール送信する
    }
}
public class UserService {
    public void register(User user) {
        EmailUtil.sendWelcome(user.getEmail());  // テストでもメールが飛ぶ
    }
}

// AFTER: インターフェース + DI
public interface EmailService { void sendWelcome(String email); }
public class UserService {
    private final EmailService emailService;
    public UserService(EmailService emailService) { this.emailService = emailService; }
    public void register(User user) { emailService.sendWelcome(user.getEmail()); }
}
// テスト時は MockEmailService を注入
```

---

## リファクタ優先度マトリクス

| パターン | リスク | テスト可能性向上 | 移行前処理としての価値 |
|---|---|---|---|
| Fat イベントハンドラ → サービス層 | 中 | 高 | 高 |
| On Error GoTo → Try/Catch | 低 | 中 | 高 |
| SQL の UI 層からの分離 | 中 | 高 | 高 |
| Module グローバル変数の解消 | 高 | 高 | 高 |
| 定数マジックナンバー除去 | 低 | 低 | 中 |
| static → DI | 中 | 高 | 中 |

---

## リファクタ計画テンプレート

```markdown
## リファクタ計画: [対象システム名]

### フェーズ 1（低リスク・即効）
- [ ] On Error GoTo → Try/Catch（機械的変換、動作変わらない）
- [ ] マジックナンバー → 定数/Enum
- [ ] 使われていないコード・コメントアウト行の削除

### フェーズ 2（中リスク・価値高）
- [ ] [フォーム名] のイベントハンドラからビジネスロジックを EmployeeService に抽出
- [ ] SQL を Repository クラスに集約
- [ ] Using ステートメントによるリソース管理の統一

### フェーズ 3（高リスク・移行前提）
- [ ] Module グローバル変数を段階的に DI に変換
- [ ] static 依存の解消（テスト可能化）

### 検証方針
- フェーズ 1 完了後: 全画面の手動動作確認（回帰テスト）
- フェーズ 2 完了後: サービス層の単体テスト追加
```

---

## Forbidden

- **「一括リファクタ」提案禁止** — 段階的に。一度に全部変えると動作確認が困難
- **テストなしのフェーズ 2 以降禁止** — サービス層抽出後は単体テストを追加してから次へ
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
