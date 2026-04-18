---
name: spec-gen-java-postgresql
description: Generates detailed design specs (詳細仕様書) for Java × PostgreSQL systems. Use when documenting Spring Boot × PostgreSQL applications, generating specs for Spring Data JPA or Hibernate repositories, documenting PostgreSQL-specific features (JSONB, sequences, RETURNING clause), or creating 詳細仕様書 for Java microservices with PostgreSQL. Covers Spring Boot 2/3, Spring Data JPA, Hibernate 6, jOOQ, and JDBC. Extends spec-gen-enterprise-ja with PostgreSQL-specific SQL patterns.
---

# Spec Generation — Java × PostgreSQL

> Java × PostgreSQL — モダンなJavaバックエンドの定番スタック。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: Scope

Ask the user:

> 対象システムを教えてください。
>
> **Java**: バージョン（8/11/17/21）/ フレームワーク（Spring Boot / Quarkus / Micronaut）
> **ORM**: Spring Data JPA / Hibernate / jOOQ / JDBC直接
> **PostgreSQL**: バージョン（12/13/14/15/16）/ 特殊機能使用（JSONB / 配列型 / UUID）
> **規模**: エンティティ数・Repository数・テーブル数

---

## PostgreSQL 固有パターン

### JSONB カラムの仕様化

```java
// Spring Data JPA × JSONB
@Entity
@Table(name = "orders")
public class Order {
    @Column(columnDefinition = "jsonb")
    @Convert(converter = JsonbConverter.class)  // カスタムコンバータ必須
    private Map<String, Object> metadata;
}
```

仕様書に記載すること:
- JSONB カラムの論理的な構造（JSONスキーマ相当）
- 検索条件に JSONB 演算子（`@>`、`->>`）を使っている場合の詳細

### RETURNING 句（INSERT後のID取得）

```java
// Spring Data JPA: save() が自動的に RETURNING を使用
// jOOQ での明示的 RETURNING
dsl.insertInto(ORDERS)
   .set(ORDERS.USER_ID, userId)
   .returning(ORDERS.ID)
   .fetchOne();

// JDBC での対応
PreparedStatement ps = conn.prepareStatement(
    "INSERT INTO orders (user_id) VALUES (?)",
    Statement.RETURN_GENERATED_KEYS
);
```

### シーケンス vs SERIAL vs GENERATED ALWAYS AS IDENTITY

```sql
-- PostgreSQL 10以降推奨: GENERATED ALWAYS AS IDENTITY
CREATE TABLE employee (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
);

-- 旧方式（非推奨だが現場では混在）
CREATE TABLE employee (
    id BIGSERIAL PRIMARY KEY  -- = SERIAL8
);
```

仕様書に IDカラムの採番方式を明記すること。

### UUID 主キー

```java
// UUID 型の主キー
@Id
@GeneratedValue(strategy = GenerationType.UUID)  // JPA 3.1 (Hibernate 6+)
private UUID id;

// または
@Id
@Column(columnDefinition = "uuid", updatable = false, nullable = false)
@GeneratedValue(generator = "UUID")
@GenericGenerator(name = "UUID", strategy = "org.hibernate.id.UUIDGenerator")
private UUID id;
```

---

## Spring Data JPA パターン

### Repository の仕様化項目

| 項目 | 記載内容 |
|---|---|
| Repository インターフェース名 | 継承元（JpaRepository / CrudRepository） |
| カスタムクエリ | `@Query` の JPQL / Native SQL |
| ページネーション | `Pageable` パラメータの有無 |
| Projection | `interface` Projection の定義 |
| N+1 対策 | `@EntityGraph` / `JOIN FETCH` の使用箇所 |

### N+1 問題の必須チェック

```java
// 危険: LAZY関連の暗黙ロード（N+1発生）
List<Order> orders = orderRepository.findAll();
orders.forEach(o -> o.getItems().size()); // N+1

// 対策1: @EntityGraph
@EntityGraph(attributePaths = {"items"})
List<Order> findAllWithItems();

// 対策2: JOIN FETCH
@Query("SELECT o FROM Order o JOIN FETCH o.items")
List<Order> findAllWithItems();
```

---

## Java × PostgreSQL 仕様書テンプレート

```markdown
## [クラス名] — 詳細仕様

### クラス概要
| 項目 | 内容 |
|---|---|
| パッケージ | com.example.repository |
| 役割 | [業務機能の説明] |
| フレームワーク | Spring Boot 3.x / Spring Data JPA |
| PostgreSQL バージョン | 15 |

### エンティティ定義（抜粋）
| フィールド | 型 | DBカラム | 制約 |
|---|---|---|---|
| id | UUID | id | PK / GENERATED ALWAYS AS IDENTITY |
| createdAt | OffsetDateTime | created_at | NOT NULL / DEFAULT NOW() |

### Repository メソッド一覧
| メソッド名 | クエリ種別 | 説明 |
|---|---|---|
| findByUserId | Derived Query | userId で検索 |
| findActiveOrders | @Query(JPQL) | status='ACTIVE' の全件取得 |

### トランザクション境界
| メソッド | @Transactional | 説明 |
|---|---|---|
| createOrder | readOnly=false | 新規注文作成（複数テーブル更新） |
| findById | readOnly=true | 読み取り専用 |
```

---

## Forbidden

- **N+1 の見落とし禁止** — LAZY関連を持つエンティティは全件チェックして記録せよ
- **SERIAL と IDENTITY の混在を無記載禁止** — 主キー採番方式を全テーブル明記
- **JSONB カラムのスキーマ無記載禁止** — 型が `jsonb` のカラムは内部構造を仕様書に記載
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
