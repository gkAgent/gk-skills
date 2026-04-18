---
name: spec-gen-java-mysql
description: Generates detailed design specs (詳細仕様書) for Java × MySQL systems — the standard stack for Java web applications. Use when documenting Spring Boot × MySQL applications, generating specs from MyBatis mapper XML files targeting MySQL, documenting Spring Data JPA × MySQL repositories, or creating 詳細仕様書 for Java web services using MySQL 5.7/8.0. Covers MySQL-specific features (LIMIT/OFFSET, AUTO_INCREMENT, TINYINT(1) boolean, ON DUPLICATE KEY UPDATE, JSON column), Spring Boot, MyBatis 3, and Spring Data JPA. Extends spec-gen-enterprise-ja.
---

# Spec Generation — Java × MySQL

> Java × MySQL — Web 系 Java バックエンドの定番スタック。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: Scope

Ask the user:

> 対象システムを教えてください。
>
> **Java**: バージョン（8/11/17/21）/ フレームワーク（Spring Boot / Spring MVC）
> **ORM**: MyBatis / Spring Data JPA / Hibernate / JDBC直接
> **MySQL**: バージョン（5.7 / 8.0）/ 文字コード（utf8mb4 推奨 / utf8）
> **規模**: クラス数・Mapperファイル数・テーブル数

---

## MySQL 固有パターン（Java 視点）

### LIMIT / OFFSET ページネーション

```xml
<!-- MyBatis: LIMIT / OFFSET ページネーション -->
<select id="findByPage" resultType="Employee">
    SELECT * FROM employee
    WHERE dept_id = #{deptId}
    ORDER BY emp_id
    LIMIT #{pageSize} OFFSET #{offset}
</select>
```

```java
// Spring Data JPA: Pageable が自動的に LIMIT/OFFSET を生成
Page<Employee> findByDeptId(Long deptId, Pageable pageable);
```

### TINYINT(1) → boolean のマッピング

```xml
<!-- MyBatis: MySQL の TINYINT(1) は boolean にマップ -->
<result property="isActive" column="is_active" javaType="boolean" jdbcType="TINYINT"/>
```

```java
// Spring Data JPA
@Column(columnDefinition = "TINYINT(1)")
private Boolean isActive;
```

仕様書に記載すること: `TINYINT(1)` カラムの論理的な意味（フラグ値の定義）

### AUTO_INCREMENT と採番方式

```java
// Spring Data JPA
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)  // MySQL AUTO_INCREMENT
private Long id;

// MyBatis: useGeneratedKeys で挿入後のIDを取得
// <insert id="insert" useGeneratedKeys="true" keyProperty="id">
```

### ON DUPLICATE KEY UPDATE（UPSERT）

```xml
<!-- MyBatis: ON DUPLICATE KEY UPDATE -->
<insert id="upsert">
    INSERT INTO user_settings (user_id, setting_key, setting_value)
    VALUES (#{userId}, #{key}, #{value})
    ON DUPLICATE KEY UPDATE
        setting_value = VALUES(setting_value),
        updated_at = NOW()
</insert>
```

仕様書に記載すること: UPSERT の条件（ユニークキー）と更新対象カラム一覧

### JSON カラム（MySQL 5.7.8+）

```java
// EF Core 相当の Spring Data JPA での JSON カラム扱い
@Column(columnDefinition = "JSON")
@Convert(converter = JsonAttributeConverter.class)
private Map<String, Object> metadata;
```

### 文字コードの罠（utf8 vs utf8mb4）

```sql
-- MySQL の utf8 は最大3バイト → 絵文字が保存できない
-- utf8mb4 が正しい UTF-8（最大4バイト）
-- 仕様書に接続文字コードとテーブルの照合順序を明記
SHOW CREATE TABLE employee;  -- CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci を確認
```

Spring Boot の設定:
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb?useUnicode=true&characterEncoding=utf8mb4&serverTimezone=Asia/Tokyo
```

---

## MyBatis × MySQL 固有チェック

| チェック項目 | 内容 |
|---|---|
| `${}` 使用箇所 | SQL インジェクション — 全件一覧化必須 |
| `useGeneratedKeys` | INSERT 後の PK 取得方式 |
| `<foreach>` の `open`/`close` | IN 句生成のカッコ設定 |
| `LIMIT` の動的組み立て | ページネーション SQL の安全な実装 |

---

## Java × MySQL 仕様書テンプレート

```markdown
## [クラス名] — 詳細仕様

### クラス概要
| 項目 | 内容 |
|---|---|
| パッケージ | com.example.mapper |
| 役割 | [業務機能の説明] |
| フレームワーク | Spring Boot 3.x / MyBatis 3.x |
| MySQL バージョン | 8.0 |
| 文字コード | utf8mb4 / utf8mb4_unicode_ci |

### SQL 仕様（Mapper: EmployeeMapper.xml）
| SQL ID | 種別 | テーブル | WHERE条件 | 特記事項 |
|---|---|---|---|---|
| findByPage | SELECT | employee | dept_id | LIMIT/OFFSET ページネーション |
| upsertSettings | INSERT | user_settings | user_id, key | ON DUPLICATE KEY UPDATE |

### 採番・フラグ定義
| カラム | 型 | Java 型 | 備考 |
|---|---|---|---|
| id | BIGINT AUTO_INCREMENT | Long | PK |
| is_active | TINYINT(1) | Boolean | 0=無効/1=有効 |
```

---

## Forbidden

- **`${}` の見落とし禁止** — MyBatis 全 Mapper XML を検索して全件記録
- **utf8 と utf8mb4 の区別を無記載禁止** — 文字コード設定を仕様書に明記
- **TINYINT(1) の意味を無記載禁止** — boolean フラグの定義値（0/1の意味）を明記
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
