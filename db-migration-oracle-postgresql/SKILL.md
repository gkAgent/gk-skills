---
name: db-migration-oracle-postgresql
description: Guides Oracle Database to PostgreSQL migration — the most common DB migration in Japan driven by Oracle licensing costs. Use when planning Oracle 11g/12c/19c to PostgreSQL migration, converting Oracle-specific SQL (ROWNUM, CONNECT BY, NVL, DECODE, SYSDATE) to PostgreSQL equivalents, migrating PL/SQL stored procedures to PL/pgSQL or application layer, assessing Oracle partitioning and sequences migration, or producing a migration plan document. Covers data type mapping, SQL syntax differences, stored procedure conversion, and tool selection (pgloader, ora2pg, AWS SCT).
---

# DB Migration — Oracle → PostgreSQL

> Oracle ライセンス費用高騰で急増する移行案件。SQL差異と型変換を完全網羅。

---

## Phase 0: Scope

Ask the user:

> 移行対象を教えてください。
>
> **Oracle**: バージョン（11g / 12c / 19c）/ エディション（SE / EE）
> **PostgreSQL**: ターゲットバージョン（14 / 15 / 16）/ ホスティング（オンプレ / RDS / Aurora / Cloud SQL）
> **規模**: テーブル数 / ストアドプロシージャ数 / View数 / トリガー数
> **アプリ言語**: Java / C# / VB.NET / TypeScript / COBOL（SQL書き換えのスコープ確認）

---

## Phase 1: データ型マッピング

| Oracle 型 | PostgreSQL 型 | 注意点 |
|---|---|---|
| `NUMBER(p,s)` | `NUMERIC(p,s)` | 精度・スケール保持。`NUMBER` 単独 → `NUMERIC` |
| `NUMBER(p,0)` | `BIGINT` / `INTEGER` | p≤9 → INTEGER, p≤18 → BIGINT |
| `VARCHAR2(n)` | `VARCHAR(n)` | 同等。バイト vs 文字の違いに注意 |
| `CHAR(n)` | `CHAR(n)` | 同等 |
| `CLOB` | `TEXT` | PostgreSQL に TEXT 長さ制限なし |
| `BLOB` | `BYTEA` | バイナリデータ。大容量は外部ストレージ推奨 |
| `DATE` | `TIMESTAMP` | **重要**: Oracle DATE は時刻含む → TIMESTAMP |
| `TIMESTAMP` | `TIMESTAMP` | 同等 |
| `TIMESTAMP WITH TIME ZONE` | `TIMESTAMPTZ` | 同等 |
| `INTERVAL` | `INTERVAL` | 同等 |
| `FLOAT` / `BINARY_FLOAT` | `DOUBLE PRECISION` | 精度注意 |
| `RAW(n)` | `BYTEA` | |
| `XMLTYPE` | `XML` | |
| `SDO_GEOMETRY` | `GEOMETRY`（PostGIS） | PostGIS 拡張必須 |

---

## Phase 2: SQL 構文変換

### ROWNUM → LIMIT / OFFSET

```sql
-- Oracle
SELECT * FROM employee WHERE ROWNUM <= 10;
SELECT * FROM (
    SELECT e.*, ROWNUM rn FROM employee e ORDER BY emp_id
) WHERE rn BETWEEN 11 AND 20;

-- PostgreSQL
SELECT * FROM employee LIMIT 10;
SELECT * FROM employee ORDER BY emp_id LIMIT 10 OFFSET 10;
```

### CONNECT BY（階層クエリ）→ WITH RECURSIVE

```sql
-- Oracle
SELECT emp_id, manager_id, emp_name, LEVEL
FROM employee
START WITH manager_id IS NULL
CONNECT BY PRIOR emp_id = manager_id;

-- PostgreSQL
WITH RECURSIVE emp_tree AS (
    SELECT emp_id, manager_id, emp_name, 1 AS level
    FROM employee WHERE manager_id IS NULL
    UNION ALL
    SELECT e.emp_id, e.manager_id, e.emp_name, t.level + 1
    FROM employee e JOIN emp_tree t ON e.manager_id = t.emp_id
)
SELECT * FROM emp_tree;
```

### DECODE → CASE WHEN

```sql
-- Oracle
SELECT DECODE(status, 'A', '有効', 'I', '無効', '不明') FROM orders;

-- PostgreSQL
SELECT CASE status WHEN 'A' THEN '有効' WHEN 'I' THEN '無効' ELSE '不明' END FROM orders;
```

### NVL / NVL2 → COALESCE / CASE

```sql
-- Oracle
SELECT NVL(commission, 0), NVL2(commission, salary + commission, salary) FROM emp;

-- PostgreSQL
SELECT COALESCE(commission, 0),
       CASE WHEN commission IS NOT NULL THEN salary + commission ELSE salary END
FROM emp;
```

### SYSDATE / SYSTIMESTAMP → NOW() / CURRENT_TIMESTAMP

```sql
-- Oracle
SELECT SYSDATE, SYSTIMESTAMP FROM DUAL;
INSERT INTO log (created_at) VALUES (SYSDATE);

-- PostgreSQL
SELECT NOW(), CURRENT_TIMESTAMP;  -- DUAL テーブル不要
INSERT INTO log (created_at) VALUES (NOW());
```

### DUAL テーブル

```sql
-- Oracle
SELECT 1 FROM DUAL;
SELECT SEQ_NAME.NEXTVAL FROM DUAL;

-- PostgreSQL: DUAL 不要
SELECT 1;
SELECT NEXTVAL('seq_name');
```

### シーケンス

```sql
-- Oracle
CREATE SEQUENCE emp_seq START WITH 1 INCREMENT BY 1;
SELECT emp_seq.NEXTVAL FROM DUAL;
INSERT INTO employee (id, name) VALUES (emp_seq.NEXTVAL, 'Tanaka');

-- PostgreSQL
CREATE SEQUENCE emp_seq START WITH 1 INCREMENT BY 1;
SELECT NEXTVAL('emp_seq');
INSERT INTO employee (id, name) VALUES (NEXTVAL('emp_seq'), 'Tanaka');
-- または IDENTITY カラム (PostgreSQL 10+)
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

### MERGE → INSERT ... ON CONFLICT

```sql
-- Oracle
MERGE INTO target t USING source s ON (t.id = s.id)
WHEN MATCHED THEN UPDATE SET t.value = s.value
WHEN NOT MATCHED THEN INSERT (id, value) VALUES (s.id, s.value);

-- PostgreSQL
INSERT INTO target (id, value) VALUES (...)
ON CONFLICT (id) DO UPDATE SET value = EXCLUDED.value;
```

---

## Phase 3: ストアドプロシージャ移行

### 移行判断マトリクス

| PL/SQL パターン | 移行先 | 難易度 |
|---|---|---|
| 単純な DML ロジック | PL/pgSQL | 低 |
| カーソル処理 | PL/pgSQL CURSOR | 中 |
| UTL_FILE（ファイルI/O） | アプリ層（Java/TS）へ移管 | 高 |
| DBMS_SCHEDULER（ジョブ） | pg_cron / アプリ cron | 中 |
| DBMS_OUTPUT.PUT_LINE | RAISE NOTICE | 低 |
| Oracle パッケージ（PACKAGE） | PostgreSQL スキーマ分割 | 中 |
| 動的SQL（EXECUTE IMMEDIATE） | EXECUTE ... USING | 中 |

### PL/SQL → PL/pgSQL 変換例

```sql
-- Oracle PL/SQL
CREATE OR REPLACE PROCEDURE update_salary(p_emp_id IN NUMBER, p_amount IN NUMBER) AS
    v_current NUMBER;
BEGIN
    SELECT salary INTO v_current FROM employee WHERE emp_id = p_emp_id;
    IF v_current IS NULL THEN
        RAISE_APPLICATION_ERROR(-20001, '社員が存在しません');
    END IF;
    UPDATE employee SET salary = salary + p_amount WHERE emp_id = p_emp_id;
    COMMIT;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RAISE_APPLICATION_ERROR(-20002, 'レコードなし');
END;

-- PostgreSQL PL/pgSQL
CREATE OR REPLACE PROCEDURE update_salary(p_emp_id BIGINT, p_amount NUMERIC)
LANGUAGE plpgsql AS $$
DECLARE
    v_current NUMERIC;
BEGIN
    SELECT salary INTO v_current FROM employee WHERE emp_id = p_emp_id;
    IF v_current IS NULL THEN
        RAISE EXCEPTION '社員が存在しません' USING ERRCODE = 'P0001';
    END IF;
    UPDATE employee SET salary = salary + p_amount WHERE emp_id = p_emp_id;
    -- PostgreSQL は自動コミット / BEGIN...COMMIT は呼び出し側で管理
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RAISE EXCEPTION 'レコードなし' USING ERRCODE = 'P0002';
END;
$$;
```

---

## Phase 4: 移行ツール

| ツール | 用途 | 備考 |
|---|---|---|
| **ora2pg** | スキーマ + データ + PL/SQL 変換 | OSSで最も実績あり |
| **pgloader** | データ移行（高速） | スキーマ変換は弱め |
| **AWS SCT** | スキーマ変換（AWS 環境） | Aurora PG ターゲット時 |
| **SQL Developer Migration** | Oracle 公式ツール | 小規模向け |

---

## Phase 5: 移行タスクリスト

```markdown
### フェーズ A: 事前調査
- [ ] Oracle 依存機能の洗い出し（CONNECT BY / MERGE / パーティション）
- [ ] PL/SQL プロシージャ数・行数カウント
- [ ] アプリ側 SQL の `${}` 文字列連結調査（SQLi + 方言依存）

### フェーズ B: スキーマ移行
- [ ] ora2pg によるDDL変換（テーブル/インデックス/シーケンス）
- [ ] DATE → TIMESTAMP 変換の影響範囲確認
- [ ] CLOB/BLOB → TEXT/BYTEA 移行判断

### フェーズ C: SQL 変換（アプリ側）
- [ ] ROWNUM → LIMIT/OFFSET 置換
- [ ] SYSDATE → NOW() / CURRENT_TIMESTAMP 置換
- [ ] NVL → COALESCE、DECODE → CASE WHEN 置換
- [ ] CONNECT BY → WITH RECURSIVE 変換（要個別対応）

### フェーズ D: PL/SQL 移行
- [ ] 単純プロシージャ → PL/pgSQL 変換
- [ ] UTL_FILE 使用箇所 → アプリ層へ移管
- [ ] DBMS_SCHEDULER → pg_cron または cron ジョブ

### フェーズ E: 検証
- [ ] ora2pg データ移行後の件数照合
- [ ] SQL 実行結果の Oracle vs PostgreSQL 比較テスト
- [ ] DATE 型の時刻部分の扱い検証
```

---

## Critical Risks

| リスク | 影響 | 対策 |
|---|---|---|
| **Oracle DATE → TIMESTAMP** | 時刻部分が切り捨てられる可能性 | アプリ全体のDATE比較ロジックを確認 |
| **暗黙的型変換** | Oracle は数値←→文字の暗黙変換が広い。PostgreSQL は厳格 | 型キャスト `::` を明示的に追加 |
| **CONNECT BY の複雑な階層** | WITH RECURSIVE への変換が難しい | 個別に移行検討。必要なら VIEW 化 |
| **Oracle パッケージの外部公開** | PostgreSQL にパッケージ概念なし | スキーマで代替またはアプリ層へ移管 |

---

## Forbidden

- **NUMBER → double/float 推奨禁止** — NUMERIC(p,s) を明記。金融データは精度損失不可
- **DATE型の Oracle/PostgreSQL 差異を無記載禁止** — 時刻部分の扱いを必ず明記
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
