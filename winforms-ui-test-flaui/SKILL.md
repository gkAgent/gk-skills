---
name: winforms-ui-test-flaui
description: Generates WinForms UI test specifications and FlaUI + xUnit test code for .NET Framework 4.8/4.8.1 WinForms applications. Use when creating UI test specs from screen designs, generating FlaUI+xUnit C# test code for WinForms forms (TextBox/Button/ComboBox/DataGridView/Label), setting up a separate test project without touching the target app, analyzing test failures with UIA tree dumps, or writing integration test scenarios across multiple forms. Works with both Claude Code and GitHub Copilot. Target apps can be VB.NET or C#; test project is always C#.
---

# WinForms UI テスト — FlaUI + xUnit ノウハウ集

> .NET Framework 4.8/4.8.1 WinForms アプリの UI テストを外付けで自動化する。
> 本体コードへの変更ゼロ。Claude Code / GitHub Copilot 両対応。
> 詳細ガイド: `products/gk-test/` の HTML ノウハウ集を参照。

---

## Phase 0: Scope（最初に確認すること）

```
対象アプリの言語:   VB.NET / C# / 混在
.NET Framework:    4.8 / 4.8.1
テスト種別:         単体UI / 結合(複数フォーム) / 失敗分析
AutomationId設定:  済み(AccessibleName) / 未設定
成果物:            テスト仕様書のみ / テストコードのみ / 両方
```

---

## Phase 1: テスト仕様書生成

**入力として受け取るもの**
- 画面設計書 または 画面スクリーンショット
- コントロール一覧（名前・種別・AutomationId）
- バリデーションルール・期待動作

**出力フォーマット（Markdown テーブル）**

```markdown
| # | 操作 | 期待結果 | 備考 |
|---|------|---------|------|
| 1 | txtEmployeeId に "EMP001" を入力 | — | — |
| 2 | btnRegister をクリック | lblStatus = "登録しました。" | 正常系 |
```

**プロンプトテンプレート（Claude Code / Copilot 共通）**

```
以下の画面仕様からFlaUI + xUnit テスト仕様書を作成してください。

## 対象画面
画面名: [画面名]
AutomationId一覧:
- [コントロール名]: [AutomationId] ([種別: TextBox/Button/Label/ComboBox])

## バリデーション・期待動作
- [条件]: [期待結果]

## 出力形式
正常系・異常系・境界値を | # | 操作 | 期待結果 | の表形式で作成。
```

---

## Phase 2: テストコード生成

**プロジェクト構成（本体を変更しない分離構成）**

```
MyApp.UITests/            ← 本体 Sln とは別フォルダ
  MyApp.UITests.csproj    ← .NET 8 + FlaUI + xUnit
  Tests/
    MainFormTests.cs
  Support/
    GkTestFixture.cs      ← アプリ起動/終了管理
    EvidenceRecorder.cs   ← スクリーンショット + レポート
```

**csproj テンプレート**

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <IsTestProject>true</IsTestProject>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="FlaUI.Core" Version="5.0.0" />
    <PackageReference Include="FlaUI.UIA3" Version="5.0.0" />
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.9.0" />
    <PackageReference Include="xunit" Version="2.9.0" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.8.2">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

**GkTestFixture テンプレート**

```csharp
public class GkTestFixture : IDisposable
{
    public Application App { get; private set; }
    public UIA3Automation Automation { get; private set; }

    public Window SetupApplication()
    {
        Automation = new UIA3Automation();
        var exePath = Environment.GetEnvironmentVariable("TARGET_APP_EXE")
            ?? FindExe("MyApp.exe");
        App = Application.Launch(exePath);
        return App.GetMainWindow(Automation, TimeSpan.FromSeconds(10));
    }

    private static string FindExe(string name)
    {
        var baseDir = AppDomain.CurrentDomain.BaseDirectory;
        var candidates = new[]
        {
            Path.Combine(baseDir, "..", "..", "..", "..", "MyApp", "bin", "Debug", name),
            Path.Combine(baseDir, "..", "..", "..", "..", "MyApp", "bin", "Release", name),
        };
        return candidates.Select(Path.GetFullPath).FirstOrDefault(File.Exists)
            ?? throw new FileNotFoundException(
                $"{name} が見つかりません。環境変数 TARGET_APP_EXE を設定してください。");
    }

    public void Dispose()
    {
        App?.Close();
        Automation?.Dispose();
    }
}
```

**プロンプトテンプレート（コード生成）**

```
以下のテスト仕様書からFlaUI + xUnit テストコードをC#で生成してください。

【前提】
- テストプロジェクト: .NET 8 xUnit
- 対象アプリ: [VB.NET/C#] .NET Framework 4.8 WinForms
- FlaUI.Core/UIA3 5.0.0
- GkTestFixture でアプリ起動・終了
- EvidenceRecorder でスクリーンショット保存 (artifacts/)
- AccessibleName が AutomationId として設定済み

【AutomationId一覧】
[コントロール一覧を貼り付け]

【テスト仕様書】
[Phase 1 で作成した Markdown テーブルを貼り付け]
```

---

## Phase 3: 失敗分析

**EvidenceRecorder が生成するレポート構造**

```
artifacts/
  TestName_YYYYMMDD_HHMMSS/
    before_failure.png   ← エラー直前のスクリーンショット
    uia_tree.txt         ← UIA ツリーダンプ（正しい AutomationId が見える）
    failure_report.md    ← AI に渡す分析依頼レポート
    test.log             ← 実行ログ
```

**プロンプトテンプレート（失敗分析）**

```
FlaUI UIテストが失敗しました。原因を分析してください。

【エラー内容】
[failure_report.md の内容を貼り付け]

【UIA ツリー（一部）】
[uia_tree.txt の内容を貼り付け]

以下を教えてください:
1. エラーの原因
2. 正しい AutomationId または操作方法
3. テストコードの修正箇所
```

---

## Phase 4: VB.NET 対象アプリへの適用

**AccessibleName の設定（Designer.vb）**

```vbnet
' Form1.Designer.vb の InitializeComponent 内に追記
Me.txtEmployeeId.AccessibleName = "txtEmployeeId"
Me.btnRegister.AccessibleName   = "btnRegister"
Me.lblStatus.AccessibleName     = "lblStatus"
' ← VS のプロパティウィンドウ「アクセシビリティ > AccessibleName」からも設定可
```

**AutomationId 未設定の場合の代替**

```csharp
// Name プロパティで検索
var btn = window.FindFirstDescendant(cf => cf.ByName("登録"));

// コントロール種別 + インデックス（最終手段）
var allTextBoxes = window.FindAllDescendants(cf => cf.ByControlType(ControlType.Edit));
```

---

## Phase 5: 大規模プロジェクトへの適用チェックリスト

- [ ] 本体 Sln とは別フォルダに UITests プロジェクトを作成した
- [ ] 本体 Sln への参照を追加していない（FlaUI の NuGet のみ）
- [ ] 環境変数 `TARGET_APP_EXE` でexeパスを外部から指定できる
- [ ] `Inspect.exe` または `FlaUI.Inspect` で AutomationId を確認した
- [ ] DPI スケールを 100% に設定した（スクリーンショット座標ずれ防止）
- [ ] artifacts/ ディレクトリを .gitignore に追加した

---

## 参考ガイド

`products/gk-test/` の HTML ノウハウ集（各章）:
- `01_setup.html` — セットアップ手順
- `02_basics.html` — FlaUI 操作パターン
- `03_spec_to_code.html` — 仕様書 → コード生成
- `04_failure_analysis.html` — 失敗 → AI 分析
- `06_integration.html` — 結合テスト
- `07_full_cycle.html` — AI フルサイクル
- `08_vbnet.html` — VB.NET 対応
