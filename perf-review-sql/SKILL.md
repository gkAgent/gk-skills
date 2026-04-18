---
name: perf-review-sql
description: SQL performance review covering N+1 queries, missing indexes, full table scans, and query plan analysis. Use when investigating slow SQL queries in Java/TypeScript/C#/VB.NET applications, reviewing DB access code for performance issues before production, analyzing ORM-generated SQL for optimization opportunities, or producing a SQL performance review report. Covers Oracle, PostgreSQL, MySQL, and SQL Server. Identifies N+1 patterns, missing composite indexes, SELECT *, implicit type conversions disabling index use, and connection pool sizing.
---

# Performance Review — SQL

> スロークエリの原因を特定し、修正案を断定する。
> N+1・インデックス漏れ・フルスキャンの3大原因を体系的に洗い出す。

---

## Activation

Ask: 対象DB（Oracle / PostgreSQL / MySQL / SQL Server）と、パフォーマンス問題の症状（スロークエリ / タイムアウト / 接続数逼迫）を教えてください。

---

## 1. N+1 クエリ（最多原因）

### 検出パターン

```java
// Java (JPA LAZY ロード)
List<Order> orders = orderRepository.findAll();          // SQL 1回
orders.forEach(o -> {
    System.out.println(o.getItems().size());             // SQL N回 → N+1
});
```

```typescript
// TypeScript (Prisma)
const orders = await prisma.order.findMany();            // SQL 1回
for (const order of orders) {
    const items = await prisma.item.findMany({           // SQL N回 → N+1
        where: { orderId: order.id }
    });
}
```

### 修正

```java
// JPA: JOIN FETCH or @EntityGraph
List<Order> orders = em.createQuery(
    "SELECT o FROM Order o JOIN FETCH o.items", Order.class).getResultList();
```

```typescript
// Prisma: include で一括取得
const orders = await prisma.order.findMany({
    include: { items: true }
});
```

---

## 2. インデックス設計

### インデックスが効かないパターン

```sql
-- パターン1: インデックスカラムへの関数適用
-- BAD: YEAR() 適用でインデックス無効
SELECT * FROM orders WHERE YEAR(created_at) = 2024;
-- GOOD: 範囲条件に変換
SELECT * FROM orders WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';

-- パターン2: 暗黙的型変換（文字列カラムに数値比較）
-- BAD: varchar の order_code に数値を渡す → 型変換でインデックス無効（MySQL/Oracle）
SELECT * FROM orders WHERE order_code = 12345;
-- GOOD:
SELECT * FROM orders WHERE order_code = '12345';

-- パターン3: LIKE の前方ワイルドカード
-- BAD: 前方一致不可でフルスキャン
SELECT * FROM products WHERE name LIKE '%キーワード%';
-- GOOD: 全文検索インデックス（FTS）を使用
-- PostgreSQL: WHERE to_tsvector('japanese', name) @@ to_tsquery('japanese', 'キーワード')

-- パターン4: OR 条件（複合インデックスの片方無効化）
-- BAD: index(dept_id, status) があっても OR でインデックスが効かない場合あり
SELECT * FROM employee WHERE dept_id = 10 OR status = 'A';
-- GOOD: UNION ALL に分解
SELECT * FROM employee WHERE dept_id = 10
UNION ALL
SELECT * FROM employee WHERE status = 'A' AND dept_id != 10;
```

### 複合インデックスの設計原則

```sql
-- 選択性の高いカラムを先頭に
-- WHERE dept_id = ? AND status = ? AND created_at > ?
-- → INDEX (dept_id, status, created_at) が有効
-- → INDEX (status, dept_id) では status の選択性が低ければ効果薄

-- カバリングインデックス（全カラムをインデックスに含める）
CREATE INDEX idx_order_cover ON orders (user_id, status) INCLUDE (amount, created_at);
-- SELECT amount, created_at FROM orders WHERE user_id = ? AND status = ?
-- → テーブルアクセスなし（インデックスのみでクエリ解決）
```

---

## 3. SELECT * の問題

```sql
-- BAD: 不要なカラムも全件取得 → ネットワーク転送量・メモリ増大
SELECT * FROM employee WHERE dept_id = 10;

-- GOOD: 必要なカラムのみ
SELECT emp_id, emp_name, salary FROM employee WHERE dept_id = 10;
```

ORM での対策:
```java
// JPA: Projection interface
interface EmployeeSummary { Long getEmpId(); String getEmpName(); }
List<EmployeeSummary> findByDeptId(Long deptId);
```

```typescript
// Prisma: select で絞り込み
const employees = await prisma.employee.findMany({
    where: { deptId },
    select: { empId: true, empName: true, salary: true }
});
```

---

## 4. 実行計画の読み方（DB別）

### PostgreSQL

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT e.emp_name, d.dept_name
FROM employee e JOIN department d ON e.dept_id = d.dept_id
WHERE e.status = 'A';

-- 要確認箇所:
-- Seq Scan → インデックスが使われていない
-- Hash Join / Nested Loop → 結合コスト確認
-- Rows=xxx (actual rows=yyy) → 統計情報の乖離（ANALYZE 実行要）
-- Buffers: shared hit=x read=y → read が多い = キャッシュミス多発
```

### Oracle

```sql
EXPLAIN PLAN FOR SELECT ...;
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY(format => 'ALL'));

-- 要確認箇所:
-- TABLE ACCESS FULL → フルスキャン
-- INDEX RANGE SCAN → インデックス使用
-- Cost: xxx → 高コストなステップを特定
```

### MySQL

```sql
EXPLAIN SELECT ...;
-- type=ALL → フルスキャン（要対処）
-- type=ref/eq_ref → インデックス使用（良好）
-- rows=xxx → 検索行数見積もり
-- Extra=Using filesort → ソートにインデックス未使用
```

---

## 5. 接続プール

```
チェック項目:
□ 接続プールサイズの設定値（デフォルト10は本番では不足することが多い）
□ connectionTimeout が短すぎてタイムアウト多発していないか
□ コネクションリーク（using なしの DbConnection など）
□ アプリのスレッド数 > プールサイズ → デッドロック的待機
```

```yaml
# HikariCP（Java/Spring Boot）推奨設定
spring:
  datasource:
    hikari:
      maximum-pool-size: 20          # CPUコア数 × 2 + 有効スピンドル数 が目安
      minimum-idle: 5
      connection-timeout: 30000      # 30秒
      idle-timeout: 600000           # 10分
      max-lifetime: 1800000          # 30分
      leak-detection-threshold: 60000 # リーク検出（本番は0=無効）
```

---

## パフォーマンスレビュー報告書

```markdown
## SQL パフォーマンスレビュー結果: [システム名]

### 重大（即修正）
| No | 分類 | 場所 | 内容 | 推定改善 |
|---|---|---|---|---|
| P001 | N+1 | OrderService.java:78 | findAll後にitemsをループ取得。JOIN FETCHに変更せよ | 100→1クエリ |
| P002 | フルスキャン | EmployeeMapper.xml | dept_id にインデックスなし。CREATE INDEX を追加せよ | 〜10倍 |

### 警告（修正推奨）
| No | 分類 | 場所 | 内容 |
|---|---|---|---|
| P003 | SELECT * | orders クエリ | 30カラム全取得。必要な6カラムのみに絞れ |
| P004 | 型変換 | WHERE order_code = #{code} | VARCHAR カラムに数値バインド。インデックス無効化の可能性 |

### 総評
[断定口調で1〜3行]
```

---

## Forbidden

- **N+1 を「パフォーマンスに影響する可能性」と曖昧記載禁止** — 発生件数（N）を推定して断定する
- **フルスキャンの見落とし禁止** — 件数の多いテーブルへの条件なし/インデックスなしSELECTを全件記録
- **途中停止禁止** / **AI voice禁止**
