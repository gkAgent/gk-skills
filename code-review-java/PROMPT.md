# コードレビュー — Java

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/code-review-java` で自動起動します（貼り付け不要）。

---

以下の指示に従って、Java コードの構造化コードレビューを実施してください。

---

## 開始時の確認

まず以下を質問してください:

> レビュー対象のクラス名・パッケージと、フレームワーク（Spring Boot / Java EE / SE）を教えてください。

---

## Java レビューチェックリスト

### A. @Transactional の落とし穴（最重要）

- **同一クラス内のメソッド呼び出し** → `@Transactional` が効かない（プロキシをバイパス）
  ```java
  // 危険: this.save() は @Transactional が動作しない
  public void process() { this.save(); }

  @Transactional
  public void save() { ... }
  ```
- `@Transactional` が `private` メソッドに付いている → 無効（記録）
- `@Transactional(readOnly = true)` で更新処理 → データが保存されない可能性
- チェック例外が `@Transactional` メソッドからスロー → デフォルトではロールバックしない（`rollbackFor` 未設定）

### B. JPA / Hibernate の N+1 問題

- `@OneToMany` / `@ManyToMany` の LAZY 関連を持つエンティティを `findAll()` 後にループ処理 → **N+1 確定**
- `@EntityGraph` または `JOIN FETCH` による解決なしに LAZY 関連へアクセス → 全件記録
- `hibernate.enable_lazy_load_no_trans=true` 設定 → **危険**: トランザクション外でのSESSION生成を隠蔽

### C. リソース管理（JDBC直接使用時）

- `Connection` / `PreparedStatement` / `ResultSet` が `try-with-resources` で囲まれているか
- `DataSource.getConnection()` を `close()` していない → コネクションリーク

### D. セキュリティ

- **SQL インジェクション**: MyBatis の `${}` 使用（`#{}` を使うべき）→ **即記録・即修正**
  ```xml
  <!-- 危険 -->
  <select id="find">SELECT * FROM T WHERE NAME = '${name}'</select>
  <!-- 安全 -->
  <select id="find">SELECT * FROM T WHERE NAME = #{name}</select>
  ```
- Spring Security: `http.csrf().disable()` → 理由なく無効化されている場合は記録
- `@PreAuthorize` / `@Secured` の漏れ → 認可なしの管理系エンドポイント
- ログ出力へのユーザー入力混入（Log Injection）

### E. 例外処理

- **チェック例外の飲み込み**: `catch (Exception e) {}` でスロー忘れ
- Spring の `@ControllerAdvice` / `@ExceptionHandler` の欠如 → 生スタックトレースがクライアントに漏れる可能性

### F. 非同期・スレッド安全

- `@Async` メソッドが `void` → 例外が握りつぶされる。`Future` / `CompletableFuture` を推奨
- `SimpleDateFormat` のインスタンス変数（スレッドアンセーフ） → `DateTimeFormatter` (Java 8+) に変更

### G. Spring Boot 固有

- `@Component` / `@Service` のフィールドインジェクション（`@Autowired` on field） → コンストラクタインジェクション推奨
- `application.properties` / `application.yml` への直接的なパスワード / APIキー記載 → 環境変数 / Secrets Manager に移行

---

## レビュー結果の出力形式

```markdown
## レビュー結果: [クラス名]

### 重大（即修正）
- [行番号] MyBatis ${} による SQLインジェクション脆弱性。#{} に変更せよ
- [行番号] @Transactional メソッド内で非チェック例外が rollbackFor 未設定。ロールバックされない

### 警告（修正推奨）
- [行番号] @OneToMany LAZY 関連を N件ループ中に参照 — N+1クエリが発生。JOIN FETCH か @EntityGraph で解決せよ
- [行番号] Connection が try-with-resources なし。リソースリークの可能性がある

### 情報
- @Transactional の同一クラス内自己呼び出しが [N]箇所。プロキシバイパスに注意
- フィールドインジェクションが [N]箇所。コンストラクタインジェクションに変更を推奨

### 総評
[断定口調で1〜3行]
```

---

## モダナイゼーション判定

| 項目 | リスク | 検出数 |
|---|---|---|
| MyBatis `${}` (SQL Injection) | 高 | N |
| `@Transactional` 自己呼び出し | 高 | N |
| N+1 クエリ（LAZY未解決） | 高 | N |
| JDBC リソースリーク | 中 | N |
| チェック例外の飲み込み | 中 | N |
| フィールドインジェクション | 低 | N |
| Java バージョン（8未満） | 高 | - |

---

## 絶対禁止事項

- **`${}` の見落とし禁止** — MyBatis 全 Mapper XML を検索して全件記録
- **@Transactional の同一クラス内呼び出しを見落とし禁止**
- **「〜の可能性があります」禁止** — 断定する
- **途中停止禁止** / **AI voice禁止**
