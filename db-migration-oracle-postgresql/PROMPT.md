# DB移行 — Oracle → PostgreSQL

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/db-migration-oracle-postgresql` で自動起動します（貼り付け不要）。

---

以下の指示に従って、Oracle から PostgreSQL への移行を支援してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: スコープの確認

まず以下を質問してください:

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
| `CLOB` | `TEXT` | PostgreSQL に TEXT 長さ制限なし |
| `BLOB` | `BYTEA` | バイナリデータ。大容量は外部ストレージ推奨 |
| `DATE` | `TIMESTAMP` | **重要**: Oracle DATE は時刻含む → TIMESTAMP |
| `TIMESTAMP WITH TIME ZONE` | `TIMESTAMPTZ` | 同等 |
| `XMLTYPE` | `XML` | |
| `SDO_GEOMETRY` | `GEOMETRY`（PostGIS） | PostGIS 拡張必須 |

---

## Phase 2: SQL 構文変換

### ROWNUM → LIMIT / OFFSET

```sql
-- Oracle
SELECT * FROM employee WHERE ROWNUM <= 10;

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

-- PostgreSQL（DUAL テーブル不要）
SELECT NOW(), CURRENT_TIMESTAMP;
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

---

## Phase 4: 移行ツール

| ツール | 用途 | 備考 |
|---|---|---|
| **ora2pg** | スキーマ + データ + PL/SQL 変換 | OSSで最も実績あり |
| **pgloader** | データ移行（高速） | スキーマ変換は弱め |
| **AWS SCT** | スキーマ変換（AWS 環境） | Aurora PG ターゲット時 |

---

## Phase 4.5: オブジェクト責務定義

移行前に全DBオブジェクトの責務と依存を洗い出してください。ora2pg の自動変換結果をそのまま本番投入することを禁止します。

```
| プロシージャ名 | 責務（1行） | 呼び出し元 | 使用Oracle固有機能 | 移行先 | 複雑度 |
|---|---|---|---|---|---|
```

---

## Phase 5: Sprint Planning — PDCAサイクル設計

Phase 4.5 の移行設計書を唯一の入力として Sprint を設計してください。

```markdown
## Sprint 0: スキーマ基盤移行
- [ ] ora2pg インストール・設定
- [ ] DATE → TIMESTAMP 変換の全テーブル影響範囲確認
- [ ] PostgreSQL ターゲット環境構築
- [ ] ora2pg データ移行後の全テーブル件数照合
```

---

## Critical Risks

| リスク | 影響 | 対策 |
|---|---|---|
| **Oracle DATE → TIMESTAMP** | 時刻部分が切り捨てられる可能性 | アプリ全体のDATE比較ロジックを確認 |
| **暗黙的型変換** | Oracle は広い。PostgreSQL は厳格 | 型キャスト `::` を明示的に追加 |
| **CONNECT BY の複雑な階層** | WITH RECURSIVE への変換が難しい | 個別に移行検討。必要なら VIEW 化 |
| **Oracle パッケージの外部公開** | PostgreSQL にパッケージ概念なし | スキーマで代替またはアプリ層へ移管 |

---

## 絶対禁止事項

- **NUMBER → double/float 推奨禁止** — NUMERIC(p,s) を明記。金融データは精度損失不可
- **DATE型の Oracle/PostgreSQL 差異を無記載禁止** — 時刻部分の扱いを必ず明記
- **途中停止禁止** / **AI voice禁止** / **工数根拠なし断言禁止**
