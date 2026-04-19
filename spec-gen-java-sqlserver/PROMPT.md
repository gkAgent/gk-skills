# Spec Generation — Java × SQL Server

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-java-sqlserver` で自動起動します（貼り付け不要）。

---

以下の指示に従って、Java × SQL Server システムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

> Azure / Windows Server 環境の Java バックエンドに。Spring Boot × SQL Server。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 対象システムを教えてください。
>
> **Java**: バージョン（8/11/17/21）/ フレームワーク（Spring Boot / Spring MVC）
> **ORM**: MyBatis / Spring Data JPA / JDBC（Microsoft JDBC Driver）
> **SQL Server**: バージョン（2019 / 2022 / Azure SQL）/ 認証（SQL Server / Windows 認証 / Azure AD）
> **規模**: クラス数・Mapperファイル数・テーブル数

---

## SQL Server 固有パターン（Java 視点）

### JDBC ドライバの確認

```xml
<!-- pom.xml: Microsoft JDBC Driver（推奨） -->
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
    <version>12.6.1.jre11</version>
</dependency>

<!-- jTDS（旧来、非推奨） -->
<!-- <groupId>net.sourceforge.jtds</groupId> -->
```

仕様書に記載すること: 使用 JDBC ドライバとバージョン（jTDS か Microsoft JDBC か）

### TOP vs OFFSET-FETCH ページネーション

```xml
<!-- MyBatis: OFFSET-FETCH ページネーション（SQL Server 2012+） -->
<select id="findByPage" resultType="Employee">
    SELECT * FROM Employee
    WHERE DeptId = #{deptId}
    ORDER BY EmpId
    OFFSET #{offset} ROWS FETCH NEXT #{pageSize} ROWS ONLY
</select>
```

```java
// Spring Data JPA: Pageable が自動的に OFFSET-FETCH に変換
Page<Employee> findByDeptId(Long deptId, Pageable pageable);
```

### IDENTITY 列と getGeneratedKeys

```xml
<!-- MyBatis: IDENTITY 列の挿入後ID取得 -->
<insert id="insert" useGeneratedKeys="true" keyProperty="id" keyColumn="Id">
    INSERT INTO Employee (Name, DeptId) VALUES (#{name}, #{deptId})
</insert>
```

### MERGE（UPSERT）

```xml
<!-- MyBatis: SQL Server MERGE 構文 -->
<update id="upsert">
    MERGE INTO UserSettings AS target
    USING (VALUES (#{userId}, #{key}, #{value})) AS src(UserId, [Key], Value)
    ON target.UserId = src.UserId AND target.[Key] = src.[Key]
    WHEN MATCHED THEN
        UPDATE SET target.Value = src.Value
    WHEN NOT MATCHED THEN
        INSERT (UserId, [Key], Value) VALUES (src.UserId, src.[Key], src.Value);
</update>
```

### Windows 認証 / Azure AD 接続設定

```yaml
# Spring Boot application.yml
spring:
  datasource:
    # SQL Server 認証
    url: jdbc:sqlserver://localhost:1433;databaseName=mydb;encrypt=true;trustServerCertificate=false
    username: sa
    password: ${DB_PASSWORD}

    # Windows 統合認証（integratedSecurity）
    # url: jdbc:sqlserver://server;databaseName=mydb;integratedSecurity=true

    # Azure AD Managed Identity
    # url: jdbc:sqlserver://server.database.windows.net;authentication=ActiveDirectoryMSI;databaseName=mydb
```

### NVARCHAR vs VARCHAR（日本語対応）

```xml
<!-- MyBatis: 日本語カラムは NVARCHAR -->
<result property="empName" column="EmpName" javaType="String" jdbcType="NVARCHAR"/>

<!-- パラメータも NVARCHAR を明示 -->
<select id="findByName">
    SELECT * FROM Employee WHERE EmpName = #{name, jdbcType=NVARCHAR}
</select>
```

---

## Java × SQL Server 仕様書テンプレート

```markdown
## [クラス名] — 詳細仕様

### クラス概要
| 項目 | 内容 |
|---|---|
| パッケージ | com.example.mapper |
| 役割 | [業務機能の説明] |
| フレームワーク | Spring Boot 3.x / MyBatis 3.x |
| SQL Server バージョン | 2022 / Azure SQL |
| JDBC ドライバ | mssql-jdbc 12.6.1 |
| 認証方式 | SQL Server 認証 / Azure AD MSI |

### SQL 仕様（Mapper: EmployeeMapper.xml）
| SQL ID | 種別 | テーブル | WHERE条件 | 特記事項 |
|---|---|---|---|---|
| findByPage | SELECT | Employee | DeptId | OFFSET-FETCH ページネーション |
| upsertSettings | MERGE | UserSettings | UserId, Key | SQL Server MERGE |

### 型マッピング
| SQL Server 型 | カラム | Java 型 | jdbcType |
|---|---|---|---|
| INT IDENTITY | Id | Long | INTEGER |
| NVARCHAR(100) | EmpName | String | NVARCHAR |
| DECIMAL(10,2) | Salary | BigDecimal | DECIMAL |
| DATETIME2 | CreatedAt | LocalDateTime | TIMESTAMP |
```

---

## 絶対禁止事項

- **jTDS ドライバの把握漏れ禁止** — jTDS と Microsoft JDBC の区別を仕様書に明記
- **NVARCHAR と VARCHAR の混在を無記載禁止** — 日本語カラムの型を明示
- **`${}` の見落とし禁止** — MyBatis 全 Mapper XML を検索して全件記録
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
