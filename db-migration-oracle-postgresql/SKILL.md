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

## Phase 4.5: オブジェクト責務定義（アセスメント補完）

**移行前に全DBオブジェクトの責務と依存を洗い出す。スキーマ変換ツールに任せきりにしない。**

### ストアドプロシージャ責務テーブル

```
| プロシージャ名 | 責務（1行） | 呼び出し元 | 使用Oracle固有機能 | 移行先 | 複雑度 |
|---|---|---|---|---|---|
| sp_update_salary | 社員給与更新 | Javaアプリ | — | PL/pgSQL | 低 |
| sp_get_dept_tree | 部門階層取得 | Javaアプリ | CONNECT BY | PL/pgSQL + WITH RECURSIVE | 高 |
| pkg_report.gen_monthly | 月次レポート生成 | バッチJCL | UTL_FILE, DBMS_OUTPUT | アプリ層へ移管 | 高 |
```

### 移行設計書

```markdown
## DB移行設計書 — [システム名]

### 1. 移行対象サマリ
- テーブル数: N（うち CLOB/BLOB含む: X件）
- ストアドプロシージャ数: N（うち移行困難: X件）
- View数: N / トリガー数: N
- **DATE型使用箇所**: 全テーブルで DATE列を洗い出し（時刻部分影響範囲）

### 2. スプリント対象の優先順位
1. スキーマ基盤（テーブル/インデックス/シーケンス）— 全体の前提
2. 単純 PL/SQL プロシージャ（Oracle固有機能なし）
3. CONNECT BY 使用プロシージャ（WITH RECURSIVE 書き換え）
4. UTL_FILE / DBMS_SCHEDULER 使用箇所（アプリ層移管）
5. トリガー（PostgreSQL トリガーへの変換 or アプリ層移管の判断）
```

---

## Phase 5: Sprint Planning — PDCAサイクル設計

**Phase 4.5 の移行設計書を唯一の入力とする。ora2pg の自動変換結果をそのまま本番投入することを禁止する。**

### 5.0 事前共通タスク（Sprint 0 — 1回のみ）

```markdown
## Sprint 0: スキーマ基盤移行

- [ ] ora2pg インストール・設定
- [ ] DDL変換: テーブル/インデックス/シーケンス → PostgreSQL DDL
- [ ] DATE → TIMESTAMP 変換の全テーブル影響範囲確認
- [ ] NUMBER → NUMERIC(p,s) / BIGINT の型マッピング確定
- [ ] PostgreSQL ターゲット環境構築（RDS Aurora / オンプレ）
- [ ] ora2pg データ移行後の全テーブル件数照合
```

### 5.1 ストアドプロシージャ単位 PDCAサイクル（Sprint N ごとに繰り返す）

```markdown
## Sprint N: [sp_xxx / pkg_xxx.procedure] → PL/pgSQL or アプリ層

### 前提（Phase 4.5 より転記）
- 責務: [1行定義]
- 使用Oracle固有機能: [CONNECT BY / UTL_FILE / DBMS_SCHEDULER / 等]
- 呼び出し元: [Javaアプリ / バッチ / 他プロシージャ]
- 移行先判断: PL/pgSQL 変換 / アプリ層移管 / pg_cron

### Step 1: 変換設計
- [ ] Oracle固有構文の PostgreSQL 対応構文を特定（Phase 2/3 変換表参照）
- [ ] `COMMIT` → 呼び出し側での transaction 管理に変更
- [ ] RAISE_APPLICATION_ERROR → RAISE EXCEPTION に変換

### Step 2: PL/pgSQL 実装（or アプリ層実装）
- [ ] PL/pgSQL 変換 or TypeScript/Java へのロジック移管
- [ ] `EXECUTE IMMEDIATE` → `EXECUTE ... USING`
- [ ] UTL_FILE → アプリ層ファイルI/O

### Step 3: 単体テスト
- [ ] 同じ引数で Oracle と PostgreSQL の戻り値/副作用が一致するか確認
- [ ] エラーケース: NO_DATA_FOUND → NOT FOUND の挙動確認

### Step 4: アプリ統合テスト
- [ ] 呼び出し元アプリで CALL → 結果が旧システムと一致するか確認
- [ ] DATE 型の時刻部分: アプリ側の比較ロジックへの影響確認
- 完了条件: Oracle と PostgreSQL で同一入力→同一出力が確認できている

### 振り返り
- Oracle固有構文で詰まった変換:
- アプリ層移管を選んだ理由:
- 次 Sprint に流用できる変換パターン:
```

### 5.2 Sprint 計画一覧

```markdown
| Sprint | 対象 | 複雑度 | 変換 | テスト | 合計 |
|--------|------|--------|------|--------|------|
| Sprint 0 | スキーマ基盤 | — | — | — | 1週 |
| Sprint 1 | 単純PL/SQL | 低 | 0.5日 | 0.5日 | 1日 |
| Sprint N | CONNECT BY含む | 高 | 2日 | 1日 | 3日 |
| Sprint X | UTL_FILE/アプリ移管 | 高 | 2日 | 2日 | 4日 |
| — | 最終全件照合 | — | — | — | 2日 |

**見積もりルール**: プロシージャ数・Oracle固有機能の数が判明した時点で更新する。今の数字は仮置き。
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
