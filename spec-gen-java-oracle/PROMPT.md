# Spec Generation — Java × Oracle

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-java-oracle` で自動起動します（貼り付け不要）。

---

以下の指示に従って、Java × Oracle システムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

> Java × Oracle — 日本SIの王道スタック。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 対象システムを教えてください。
>
> **Java**: バージョン（8/11/17/21）/ フレームワーク（Spring Boot / Spring MVC / Java EE / SE）
> **ORM**: MyBatis / Spring Data JPA / Hibernate / JDBC直接
> **Oracle**: バージョン（11g / 12c / 19c）/ 接続方式（JDBC / UCP / JNDI）
> **規模**: クラス数・Mapperファイル数・テーブル数

---

## Oracle 固有パターン

### ROWNUM vs ROW_NUMBER()

```sql
-- Oracle 11g 以前: ROWNUM によるページネーション
SELECT * FROM (
    SELECT a.*, ROWNUM rn FROM (
        SELECT * FROM EMPLOYEE ORDER BY EMP_ID
    ) a WHERE ROWNUM <= 20
) WHERE rn > 10;

-- Oracle 12c 以降: FETCH FIRST / OFFSET
SELECT * FROM EMPLOYEE
ORDER BY EMP_ID
OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY;
```

仕様書に記載すること:
- どちらのページネーション方式を使っているか
- `ROWNUM` を使っている場合、Oracle 12c 以降への移行方針

### DATE 型の罠

Oracle の `DATE` 型は **時刻（HH:MM:SS）を含む**。
ANSI SQL の `DATE`（日付のみ）と異なる点を必ず明記。

```java
// MyBatis の TypeHandler 設定が必要な場合あり
// DATE 型を java.util.Date で扱うと時刻情報を保持
// 日付のみなら java.sql.Date または LocalDate を推奨
```

### NUMBER 型の精度

```sql
NUMBER(10, 2)  -- → Java の BigDecimal を推奨（double/float は精度損失）
NUMBER(38)     -- → Long または BigInteger
```

MyBatis mapper XML での設定:
```xml
<result property="amount" column="AMOUNT" javaType="java.math.BigDecimal" jdbcType="NUMERIC"/>
```

---

## MyBatis パターン

### Mapper XML の仕様化項目

| 項目 | 記載内容 |
|---|---|
| `namespace` | 対応する Mapper インターフェース FQN |
| `resultMap` | DBカラム名 → Javaプロパティ名のマッピング一覧 |
| `<if>` / `<choose>` | 動的SQLの条件分岐ロジック |
| `#{}` vs `${}` | `#{}` = PreparedStatement（推奨）/ `${}` = 文字列連結（SQLインジェクション注意） |
| `<foreach>` | IN句の生成パターン（パラメータ型・区切り文字） |

### N+1 問題の記録

```xml
<!-- N+1 が発生するパターン — 仕様書に記録して対策を明記 -->
<select id="findOrders" resultType="Order">
    SELECT * FROM ORDERS WHERE USER_ID = #{userId}
</select>
<!-- この後に各 Order に対して Item を個別SELECT → N+1 -->

<!-- 対策: JOIN or 別クエリでの一括取得 -->
```

---

## Spring Boot × Oracle 接続設定の仕様化

```yaml
# application.yml の接続設定を仕様書に記録
spring:
  datasource:
    url: jdbc:oracle:thin:@//hostname:1521/servicename
    driver-class-name: oracle.jdbc.OracleDriver
    hikari:
      maximum-pool-size: 10         # 接続プールサイズ
      connection-timeout: 30000     # 接続タイムアウト
      idle-timeout: 600000          # アイドルタイムアウト
```

記載項目:
- 接続方式（SID / サービス名 / TNS）
- 接続プール設定（HikariCP / UCP）
- JNDI 使用の場合はアプリサーバの設定先

---

## Java × Oracle 仕様書テンプレート

```markdown
## [クラス名] — 詳細仕様

### クラス概要
| 項目 | 内容 |
|---|---|
| パッケージ | com.example.service |
| 役割 | [業務機能の説明] |
| フレームワーク | Spring Boot 3.x / MyBatis 3.x |
| Oracle バージョン | 19c |

### メソッド一覧
| メソッド名 | 引数 | 戻り値 | 処理概要 |
|---|---|---|---|
| findByEmpId | empId: Long | Optional<Employee> | 社員ID検索 |

### SQL 仕様（Mapper: EmployeeMapper.xml）
| SQL ID | 種別 | テーブル | WHERE条件 |
|---|---|---|---|
| findByEmpId | SELECT | EMPLOYEE | EMP_ID = #{empId} |

### 例外処理
| 例外クラス | 発生条件 | 処理内容 |
|---|---|---|
| DataAccessException | DB接続失敗 | ログ出力してカスタム例外に変換 |
```

---

## 絶対禁止事項

- **`${}` の見落とし禁止** — SQLインジェクション脆弱性。Mapper XML の全 `${}` を一覧化せよ
- **NUMBER → double/float 禁止** — 金融計算は BigDecimal を明記
- **ROWNUM の Oracle バージョン依存を無視禁止** — 12c 以降の FETCH FIRST との差異を明記
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
