---
name: code-review-enterprise-ja
description: Performs structured code review for Japanese enterprise systems using language-specific checklists. Supports Java (8/11/17/21), TypeScript, C#/.NET, VB.NET, COBOL. Use when a user wants to review source code before deployment, audit legacy code for quality or security, review a pull request, conduct pre-migration code assessment, or generate a review report in Markdown. Follows the 5 D-principles: question-driven start, full-coverage traversal, Files>Prompts, 80%-NOT instructions, no AI-voice filler.
---

# Code Review Kit — Enterprise Japanese Edition

## Activation

When this skill is invoked, follow the exact flow below.

---

## Phase 0: Identify Target

Ask the user:

> レビュー対象を教えてください。
>
> - **言語**: Java / TypeScript / C# / VB.NET / COBOL / その他
> - **対象**: ファイル名 or 機能名 or PR番号
> - **観点**: セキュリティ / 品質 / パフォーマンス / 全部

---

## Phase 1: Coverage Plan (D2)

List ALL files or functions to review before starting. Update status after each.

```
## レビュー踏破計画

| # | ファイル/関数 | 観点 | ステータス |
|---|---|---|---|
| 1 | [ファイル名] | 全般 | [ ] 未着手 |
```

Do not skip files. Do not prioritize by your own judgment.

---

## Phase 2: Language-Specific Review

### Java Review Checklist

**A. バージョン・依存関係**
- Java バージョン（8/11/17/21）と`pom.xml`/`build.gradle`の`sourceCompatibility`が一致しているか
- Spring Boot 2.7以下（サポート終了）、Log4j 1.x/2.x (<2.17.1)が混入していないか
- `-Xlint:all` でraw type/unchecked警告がないか

**B. Null安全**
- 外部入力・DB戻り値のnull考慮
- `Optional`をフィールドや引数に使っていないか（戻り値専用）
- `Optional.get()`を`isPresent`チェックなしに使っていないか

**C. 例外処理**
- `catch(Exception e)`で飲み込んでいないか
- チェック例外の連鎖（`throw new XxxException("msg", e)`）
- トランザクション境界と例外の関係（`@Transactional(rollbackFor=)`）

**D. セキュリティ**
- SQLインジェクション（文字列連結SQL禁止、PreparedStatement/NamedParameterJdbcTemplate必須）
- XSS（Thymeleaf `th:text` vs `th:utext`、JSP `<c:out>`）
- CSRF対策（Spring Security の`csrf()` 有効か）
- 認証・認可漏れ（`@PreAuthorize`、`SecurityContextHolder`の使い方）

**E. Oracle固有（該当時）**
- `CallableStatement`でのPackage呼び出し: IN/OUTパラメータのバインド漏れ
- `SYS_REFCURSOR`のクローズ忘れ
- `OracleConnection`のコネクションリーク

---

### TypeScript Review Checklist

**A. 型安全**
- `any`型の使用箇所（`unknown`への置き換え候補）
- `as`キャストの乱用（型ガード関数への置き換え）
- `!`（Non-null assertion）の必要性

**B. 非同期**
- `async/await`に対応した`try/catch`があるか
- Promiseチェーンの`.catch()`漏れ
- `Promise.all`使用時のエラー処理

**C. セキュリティ**
- `dangerouslySetInnerHTML`の使用（XSSリスク）
- 環境変数の`process.env`直参照（サーバーサイドのみ許可）
- SQLインジェクション（ORMのraw queryの文字列連結）

**D. React固有（該当時）**
- `useEffect`の依存配列漏れ
- コンポーネントの巨大化（200行超は分割検討）
- `key`プロップの`index`使用（並び替えがある場合はNG）

---

### C# / VB.NET Review Checklist

**A. リソース管理**
- `IDisposable`実装クラスを`using`で囲んでいるか
- `SqlConnection`/`SqlCommand`のリーク

**B. 例外処理**
- `catch(Exception)`でスタックトレースを飲み込んでいないか
- `throw ex`（スタックトレース破壊）vs `throw`（正しい再スロー）

**C. セキュリティ**
- SQLインジェクション（`SqlCommand.Parameters`を使っているか）
- `ViewState`の暗号化・MAC検証（WebForms）
- `Authorize`属性の付け忘れ（ASP.NET MVC/Core）

**D. VB.NET固有**
- `Option Strict On`が宣言されているか
- `IsNothing()`と`Is Nothing`の混在（統一すること）
- レイトバインディング（`Object`型への代入）

---

### COBOL Review Checklist

**A. データ定義**
- `WORKING-STORAGE`のPICサイズがDB列定義と一致しているか
- `COMP-3`（パック10進）の桁数とデータ長の整合

**B. SQL**
- すべての`EXEC SQL`の後に`SQLCODE`チェックがあるか
- `NOT FOUND`（SQLCODE +100）の明示的ハンドリング
- `CURSOR`のOPEN/FETCH/CLOSEが対称になっているか

**C. ファイル操作**
- `OPEN`後の`FILE STATUS`確認
- `CLOSE`の漏れ（GOTOによる制御フロー中断時）

---

## Phase 3: Review Report Output

For each file/function, output:

```markdown
## レビュー結果: [ファイル名]

### 重大（即修正）
- [行番号] [指摘内容] — [修正方針]

### 警告（修正推奨）
- [行番号] [指摘内容]

### 情報（認識しておくべき事項）
- [内容]

### 総評
[1〜3行。断定口調で。「〜と思われます」禁止]
```

---

## Forbidden — 絶対禁止事項 (D4)

- **あいまい判定禁止**: 「〜の可能性があります」ではなく「〜である。修正せよ」と断定する
- **途中停止禁止**: 踏破計画の全ファイルをレビューする前に止まらない
- **AI voice 禁止**: 「コードの品質向上のために〜」などの冗長な導入文を書かない
- **推測でのセキュリティ指摘禁止**: コードに根拠がないセキュリティ問題を指摘しない
