---
name: security-review-web
description: Security-focused code review against OWASP Top 10 (2021) for web applications. Use when conducting security review of Java/TypeScript/C#/VB.NET web applications before production release, performing vulnerability assessment of REST APIs, reviewing authentication and authorization implementation, or producing a security review report for audit purposes. Covers OWASP A01-A10 (2021): Broken Access Control, Injection, Cryptographic Failures, Insecure Design, Security Misconfiguration, Vulnerable Components, Auth Failures, SSRF, and more.
---

# Security Review — Web Application (OWASP Top 10)

> コードレビューとは別軸の脆弱性診断。OWASP Top 10 を系統的にチェックする。
> 言語固有のコード品質は **code-review-[言語]** スキルを使うこと。

---

## Activation

Ask: 対象システムを教えてください。**言語/FW**・**認証方式**（セッション/JWT/OAuth）・**公開範囲**（社内/インターネット公開）

---

## OWASP Top 10 (2021) チェックリスト

### A01: Broken Access Control（アクセス制御の不備）— 最多リスク

```
チェック項目:
□ 認証なしでアクセスできるエンドポイントの洗い出し
□ ユーザーAのデータをユーザーBが取得できるか（IDOR: 直接オブジェクト参照）
□ 管理者機能への一般ユーザーアクセス制限
□ JWT/セッション失効後のアクセス拒否確認
□ CORS 設定（AllowAnyOrigin + AllowCredentials の組み合わせ禁止）
```

```typescript
// 危険: ユーザーIDをリクエストパラメータから取得（IDOR）
app.get("/api/orders/:userId", async (req, res) => {
    const orders = await getOrders(req.params.userId);  // 任意のユーザーIDを指定可能
    res.json(orders);
});

// 正しい: セッション/JWTから認証済みユーザーIDを取得
app.get("/api/orders", async (req, res) => {
    const userId = req.user.id;  // 認証済みIDのみ使用
    const orders = await getOrders(userId);
    res.json(orders);
});
```

### A02: Cryptographic Failures（暗号化の失敗）

```
チェック項目:
□ パスワードのハッシュアルゴリズム（MD5/SHA1 → bcrypt/Argon2 に変更必須）
□ 機密データの平文保存（DB カラム、ログ出力）
□ HTTP での機密データ送信（HTTPS 強制確認）
□ 弱い暗号化アルゴリズム（DES/3DES/RC4 使用）
□ ハードコードされた暗号鍵・IV
```

```java
// 危険: MD5 パスワードハッシュ
String hash = DigestUtils.md5Hex(password);

// 正しい: BCrypt
String hash = BCrypt.hashpw(password, BCrypt.gensalt(12));
boolean match = BCrypt.checkpw(inputPassword, storedHash);
```

### A03: Injection（インジェクション）

SQLi / LDAP Injection / OS Command Injection / SSTI

```
チェック項目:
□ SQL: 文字列連結・補間によるクエリ生成
□ SQL: ORM の Raw クエリへのユーザー入力混入
□ OS コマンド: exec/system/spawn へのユーザー入力混入
□ LDAP: フィルタ文字列へのユーザー入力混入
□ テンプレートエンジン: ユーザー入力のテンプレートとしての評価
```

```java
// 危険: 文字列連結SQL（SQLi）
String sql = "SELECT * FROM users WHERE name = '" + userName + "'";

// 正しい: PreparedStatement / MyBatis #{}
PreparedStatement ps = conn.prepareStatement(
    "SELECT * FROM users WHERE name = ?");
ps.setString(1, userName);
```

```typescript
// 危険: OS コマンドインジェクション
import { exec } from "child_process";
exec(`convert ${req.body.filename} output.pdf`);  // ファイル名に ; rm -rf / など混入可能

// 正しい: ユーザー入力をコマンドに混入しない。ファイル名はホワイトリスト検証
```

### A04: Insecure Design（安全でない設計）

```
チェック項目:
□ レート制限なしのログイン試行（ブルートフォース対策）
□ パスワードリセットフローの予測可能なトークン
□ 機密操作（振込・削除）の再認証なし
□ ビジネスロジックの整合性チェック欠如（数量に負値を指定可能など）
```

### A05: Security Misconfiguration（セキュリティの設定ミス）

```
チェック項目:
□ デフォルト認証情報（admin/admin 等）の残存
□ スタックトレースのクライアントへの露出
□ デバッグモードの本番稼働
□ 不要な HTTP メソッドの許可（TRACE/TRACK）
□ セキュリティヘッダーの欠如（CSP / X-Frame-Options / HSTS）
□ CORS の過剰な許可（AllowAnyOrigin）
```

```typescript
// 推奨セキュリティヘッダー（helmet.js / カスタムミドルウェア）
app.use((req, res, next) => {
    res.setHeader("X-Frame-Options", "DENY");
    res.setHeader("X-Content-Type-Options", "nosniff");
    res.setHeader("Strict-Transport-Security", "max-age=31536000; includeSubDomains");
    res.setHeader("Content-Security-Policy", "default-src 'self'");
    next();
});
```

### A06: Vulnerable and Outdated Components（脆弱なコンポーネント）

```
チェック項目:
□ 既知脆弱性のある依存ライブラリ（npm audit / mvn dependency-check / dotnet list package --vulnerable）
□ サポート終了ランタイム（Java 8 LTS切れ / Node.js EOL）
□ log4j 2.x（Log4Shell CVE-2021-44228）の残存
□ Spring Framework 古バージョン（Spring4Shell CVE-2022-22965）
```

```bash
# 依存脆弱性チェックコマンド
npm audit --audit-level=high        # Node.js
mvn dependency-check:check          # Maven (OWASP plugin)
dotnet list package --vulnerable    # .NET
```

### A07: Identification and Authentication Failures（認証の失敗）

```
チェック項目:
□ JWT の署名検証なし / alg=none 許可
□ セッションIDの URL パラメータ渡し
□ セッション固定攻撃対策（ログイン後のセッションID再生成）
□ 無制限のセッション有効期間
□ パスワード強度チェックなし（最小長・複雑性）
```

```typescript
// 危険: JWT の署名検証なし
const payload = JSON.parse(atob(token.split(".")[1]));  // 署名未検証

// 正しい: 署名検証
import jwt from "jsonwebtoken";
const payload = jwt.verify(token, process.env.JWT_SECRET!);
```

### A08: Software and Data Integrity Failures（ソフトウェアとデータの整合性の失敗）

```
チェック項目:
□ CI/CD パイプラインへの不正アクセス防止
□ npm/Maven パッケージの整合性検証（lockfile の使用）
□ デシリアライズ処理でのユーザー入力（Java ObjectInputStream / PHP unserialize）
□ 署名なしアップデートの自動適用
```

### A09: Security Logging and Monitoring Failures（セキュリティログとモニタリングの失敗）

```
チェック項目:
□ ログイン成功・失敗の記録（ユーザーID・IP・タイムスタンプ）
□ 認可失敗の記録（403 ログ）
□ パスワード・トークンのログ出力（ログインジェクション）
□ 障害・異常の検知と通知の仕組み
```

```java
// 危険: パスワードのログ出力
log.info("Login attempt: user={}, password={}", username, password);

// 正しい: パスワード出力禁止
log.info("Login attempt: user={}", username);
log.warn("Login failed: user={}, ip={}", username, request.getRemoteAddr());
```

### A10: Server-Side Request Forgery (SSRF)

```
チェック項目:
□ ユーザー入力 URL への HTTP リクエスト発行
□ ファイルパスへのユーザー入力混入（パストラバーサル）
□ 内部ネットワーク（169.254.x.x / 10.x.x.x）へのアクセス制限
□ リダイレクト先 URL のバリデーション
```

---

## セキュリティレビュー報告書

```markdown
## セキュリティレビュー結果: [システム名]
**実施日**: YYYY-MM-DD
**対象**: [対象ファイル・コンポーネント]

### 重大（即修正）
| No | 分類 | 場所 | 内容 | OWASP |
|---|---|---|---|---|
| S001 | SQLインジェクション | UserService.java:45 | 文字列連結SQL。PreparedStatementに変更せよ | A03 |
| S002 | 認証バイパス | /api/admin/* | [Authorize]なし。全エンドポイントに認証を追加せよ | A01 |

### 警告（修正推奨）
| No | 分類 | 場所 | 内容 | OWASP |
|---|---|---|---|---|
| S003 | 弱いパスワードハッシュ | AuthService.java:22 | MD5使用。BCrypt(cost=12)に移行せよ | A02 |

### 情報
- セキュリティヘッダー未設定（X-Frame-Options等）— helmetまたは手動設定を推奨
- npm audit で [N]件の脆弱性（高: N件）— 要更新

### 総評
[断定口調で1〜3行]
```

---

## Forbidden

- **IDOR の確認漏れ禁止** — 全リソース取得APIでユーザーIDを認証トークンから取得しているか必ず確認
- **MD5/SHA1 パスワードハッシュの見落とし禁止** — 即記録・即修正指示
- **「〜の可能性があります」禁止** — 断定する
- **途中停止禁止** / **AI voice禁止**
