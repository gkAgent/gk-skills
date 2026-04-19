# コードレビュー — エンタープライズ版

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/code-review-enterprise` で自動起動します（貼り付け不要）。

---

以下の指示に従って、エンタープライズシステムの構造化コードレビューを実施してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: レビュー対象の特定

まず以下を質問してください:

> Please tell me what you want to review.
>
> - **Language**: Java / TypeScript / C# / VB.NET / COBOL / Other
> - **Target**: File name(s), feature name, or PR number
> - **Focus**: Security / Quality / Performance / All of the above

---

## Phase 1: 踏破計画

レビュー開始前に全ファイル・関数を列挙し、完了後にステータスを更新してください。

```
## Review Coverage Plan

| # | File / Function | Focus | Status |
|---|---|---|---|
| 1 | [file name] | General | [ ] Not started |
```

ファイルをスキップしないこと。自己判断で優先順位をつけないこと。

---

## Phase 2: 言語別レビュー

### Java レビューチェックリスト

**A. バージョン・依存関係**
- Java バージョンと`pom.xml`/`build.gradle`の`sourceCompatibility`が一致しているか
- Spring Boot 2.7以下（サポート終了）、Log4j 1.x/2.x (<2.17.1)が混入していないか

**B. Null安全**
- 外部入力・DB戻り値のnull考慮
- `Optional.get()`を`isPresent`チェックなしに使っていないか

**C. 例外処理**
- `catch(Exception e)`で飲み込んでいないか
- トランザクション境界と例外の関係（`@Transactional(rollbackFor=)`）

**D. セキュリティ**
- SQLインジェクション（文字列連結SQL禁止、PreparedStatement必須）
- XSS（Thymeleaf `th:text` vs `th:utext`）
- CSRF対策（Spring Security の`csrf()` 有効か）

**E. Oracle固有（該当時）**
- `CallableStatement`でのPackage呼び出し: IN/OUTパラメータのバインド漏れ
- `SYS_REFCURSOR`のクローズ忘れ

---

### TypeScript レビューチェックリスト

**A. 型安全**
- `any`型の使用箇所（`unknown`への置き換え候補）
- `as`キャストの乱用（型ガード関数への置き換え）

**B. 非同期**
- `async/await`に対応した`try/catch`があるか
- `Promise.all`使用時のエラー処理

**C. セキュリティ**
- `dangerouslySetInnerHTML`の使用（XSSリスク）
- SQLインジェクション（ORMのraw queryの文字列連結）

**D. React固有（該当時）**
- `useEffect`の依存配列漏れ
- `key`プロップの`index`使用（並び替えがある場合はNG）

---

### C# / VB.NET レビューチェックリスト

**A. リソース管理**
- `IDisposable`実装クラスを`using`で囲んでいるか
- `SqlConnection`/`SqlCommand`のリーク

**B. 例外処理**
- `catch(Exception)`でスタックトレースを飲み込んでいないか
- `throw ex`（スタックトレース破壊）vs `throw`（正しい再スロー）

**C. セキュリティ**
- SQLインジェクション（`SqlCommand.Parameters`を使っているか）
- `Authorize`属性の付け忘れ（ASP.NET MVC/Core）

**D. VB.NET固有**
- `Option Strict On`が宣言されているか
- レイトバインディング（`Object`型への代入）

---

### COBOL レビューチェックリスト

**A. データ定義**
- `WORKING-STORAGE`のPICサイズがDB列定義と一致しているか
- `COMP-3`（パック10進）の桁数とデータ長の整合

**B. SQL**
- すべての`EXEC SQL`の後に`SQLCODE`チェックがあるか
- `CURSOR`のOPEN/FETCH/CLOSEが対称になっているか

**C. ファイル操作**
- `OPEN`後の`FILE STATUS`確認
- `CLOSE`の漏れ（GOTOによる制御フロー中断時）

---

## Phase 3: レビュー結果の出力形式

```markdown
## Review Result: [File Name]

### Critical (Fix immediately)
- [Line number] [Finding] — [Remediation approach]

### Warning (Fix recommended)
- [Line number] [Finding]

### Info (Awareness items)
- [Content]

### Summary
[1–3 sentences. Use direct, assertive language. "It appears that…" is prohibited]
```

---

## 絶対禁止事項

- **あいまい判定禁止**: 「〜の可能性があります」ではなく「〜である。修正せよ」と断定する
- **途中停止禁止**: 踏破計画の全ファイルをレビューする前に止まらない
- **AI voice 禁止**: 冗長な導入文を書かない
- **推測でのセキュリティ指摘禁止**: コードに根拠がないセキュリティ問題を指摘しない
