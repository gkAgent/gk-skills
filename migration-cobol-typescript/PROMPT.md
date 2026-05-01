# Migration — COBOL → TypeScript / Node.js

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/migration-cobol-typescript` で自動起動します（貼り付け不要）。

---

以下の指示に従って、COBOL レガシーシステムの TypeScript/Node.js 移行を支援してください。フェーズを順番に進め、スキップしないでください。

> The hardest migration. But someone has to do it.
> COBOL → DB2 仕様書生成は **spec-gen-cobol-db2** を使うこと。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 移行対象を教えてください。
>
> **現行**: COBOL ランタイム / 実行環境（JCLバッチ / CICS / IMS）
> **移行先**: TypeScript + Node.js バッチ / NestJS マイクロサービス / Next.js API
> **DB**: DB2 for z/OS / DB2 LUW → PostgreSQL / MySQL / SQL Server
> **規模**: PGM数 / COPY ブック数 / JCLジョブ数

---

## Phase 1: アセスメント

### 1.1 難易度マトリクス

| COBOL パターン | TypeScript 対応 | 難易度 |
|---|---|---|
| 線形バッチ処理（PERFORM UNTIL EOF） | Node.js ストリーム / バッチループ | 低 |
| EXEC SQL（SELECT/INSERT/UPDATE） | Prisma / pg パラメータクエリ | 低〜中 |
| WORKING-STORAGE の数値計算 | TypeScript number / BigInt | 中 |
| `COMP-3`（パック10進） | `BigDecimal` ライブラリ / 文字列処理 | 中 |
| `OCCURS`（配列） | TypeScript 配列 | 低 |
| `REDEFINES`（オーバーレイ） | TypeScript Union Type / Buffer 解析 | 高 |
| `CALL` 外部プログラム | モジュール import / マイクロサービス | 中 |
| CICS トランザクション | REST API エンドポイント | 高 |
| `EXEC CICS RECEIVE MAP` (BMS) | React/HTML フォーム | 高（UI再設計） |
| ファイル I/O (VSAM/順編成) | Node.js `fs` / ストリーム | 中 |
| JCL ジョブスケジュール | cron / BullMQ / Temporal | 中 |

### 1.2 移行不可・要判断

- MQ Series / IBM MQ → RabbitMQ / Kafka への置き換え検討
- IMS 階層DB → RDB への再設計（データモデルの根本変更）
- COBOL の10進固定小数点演算（金融計算） → `decimal.js` / `big.js` で誤差を抑える
- `SORT` / `MERGE` ステートメント → DB ORDER BY またはメモリソート

---

## Phase 2: 変換パターン集

### 2.1 PICTURE clause → TypeScript 型

```cobol
* COBOL WORKING-STORAGE
01 WS-EMPLOYEE.
   05 WS-EMP-ID      PIC 9(6).           " → number (integer)
   05 WS-EMP-NAME    PIC X(40).          " → string (max 40 chars)
   05 WS-SALARY      PIC 9(9)V99 COMP-3. " → Decimal (精度注意)
   05 WS-HIRE-DATE   PIC 9(8).           " → string "YYYYMMDD" → Date
   05 WS-IS-ACTIVE   PIC X(1).           " → boolean ('Y'/'N')
```
```typescript
// TypeScript
interface Employee {
    empId: number;           // PIC 9(6)
    empName: string;         // PIC X(40) — trim trailing spaces
    salary: Decimal;         // PIC 9(9)V99 COMP-3 — use decimal.js
    hireDate: Date;          // PIC 9(8) — parse "YYYYMMDD"
    isActive: boolean;       // PIC X(1) — 'Y' → true
}
```

### 2.2 OCCURS（配列）

```cobol
01 WS-ORDER-TABLE.
   05 WS-ORDER OCCURS 50 TIMES.
      10 WS-ORDER-ID  PIC 9(8).
      10 WS-AMOUNT    PIC 9(7)V99.
```
```typescript
interface Order {
    orderId: number;
    amount: Decimal;
}
const orderTable: Order[] = [];  // 動的サイズ（最大50件の制約はバリデーションで）
```

### 2.3 REDEFINES（共用体）

```cobol
01 WS-DATA           PIC X(10).
01 WS-DATA-AS-NUM REDEFINES WS-DATA PIC 9(10).
```
```typescript
// TypeScript Union Type
type DataRecord =
    | { type: "text"; value: string }
    | { type: "number"; value: number };

// または Buffer で生バイト解析
function parseRecord(buf: Buffer): DataRecord {
    const text = buf.toString("ascii").trimEnd();
    const num = parseInt(buf.toString("ascii"), 10);
    return isNaN(num) ? { type: "text", value: text } : { type: "number", value: num };
}
```

### 2.4 バッチループ

```cobol
PROCEDURE DIVISION.
    OPEN INPUT IN-FILE OUTPUT OUT-FILE
    PERFORM UNTIL EOF-FLAG = 'Y'
        READ IN-FILE
            AT END MOVE 'Y' TO EOF-FLAG
            NOT AT END PERFORM PROCESS-RECORD
        END-READ
    END-PERFORM
    CLOSE IN-FILE OUT-FILE
    STOP RUN.
```
```typescript
// Node.js readline ストリーム
import { createReadStream } from "fs";
import { createInterface } from "readline";

async function processBatch(inputPath: string): Promise<void> {
    const rl = createInterface({
        input: createReadStream(inputPath, { encoding: "ascii" }),
    });
    for await (const line of rl) {
        await processRecord(line);
    }
}
```

### 2.5 EXEC SQL → Prisma / pg

```cobol
EXEC SQL
    SELECT EMP_NAME, SALARY
    INTO :WS-EMP-NAME, :WS-SALARY
    FROM EMPLOYEE
    WHERE EMP_ID = :WS-EMP-ID
END-EXEC.
IF SQLCODE = +100
    MOVE 'NOT FOUND' TO WS-STATUS
END-IF.
```
```typescript
// Prisma
const employee = await prisma.employee.findUnique({
    where: { empId: wsEmpId },
    select: { empName: true, salary: true },
});
if (!employee) {
    wsStatus = "NOT FOUND";
    return;
}

// pg (raw SQL)
const result = await pool.query<{ emp_name: string; salary: string }>(
    "SELECT emp_name, salary FROM employee WHERE emp_id = $1",
    [wsEmpId],
);
if (result.rows.length === 0) { wsStatus = "NOT FOUND"; return; }
```

### 2.6 JCL → Node.js + cron

```jcl
//JOBBATCH JOB (ACCT),'DAILY BATCH',CLASS=A
//STEP1   EXEC PGM=DAILYRPT
//INFILE  DD DSN=PROD.INPUT.FILE,DISP=SHR
//OUTFILE DD DSN=PROD.OUTPUT.FILE,DISP=(NEW,CATLG)
```
```typescript
// Node.js batch (TypeScript)
// jobs/daily-report.ts
export async function runDailyReport(): Promise<void> {
    const inputPath = process.env.INPUT_FILE_PATH!;
    const outputPath = process.env.OUTPUT_FILE_PATH!;
    await processBatch(inputPath, outputPath);
}

// cron/scheduler.ts (node-cron)
import cron from "node-cron";
cron.schedule("0 2 * * *", () => runDailyReport().catch(console.error));
```

---

## Phase 1.5: プログラム責務定義（アセスメント補完）

**COBOL に Designer ファイルは存在しない。代わりに PGM 単位の責務と依存関係を漏らさず洗い出す。省略禁止。**

### PGM 責務テーブル（全 PGM 分作成）

```
| PGM名 | DIVISION | 責務（1行） | 呼び出し先 CALL | COPY ブック | 複雑度 |
|---|---|---|---|---|---|
| DAILYRPT | PROCEDURE | 日次売上集計 → CSV出力 | CALCUTIL, FILEIO | EMPDEF, ORDDEF | 中 |
| CALCUTIL | PROCEDURE | 金額計算ユーティリティ（COMP-3） | — | AMTDEF | 低 |
| CICSORD | CICS | 受注照会オンライン処理 | CALCUTIL | ORDDEF | 高 |
```

### JCL ジョブ依存テーブル

```
| ジョブ名 | ステップ順 | PGM | 入力データセット | 出力先 | 実行頻度 |
|---|---|---|---|---|---|
| JOBBATCH | STEP1→STEP2→STEP3 | DAILYRPT→CALCUTIL→RPTOUT | PROD.INPUT.FILE | PROD.REPORT | 毎日 02:00 |
```

---

## Phase 1.6: 分析結果 → 移行設計書

**Phase 1 + 1.5 の結果を「移行設計書」にまとめる。これが以降の全実装の唯一の根拠となる。**

```markdown
## 移行設計書 — [システム名]

### 1. 移行対象サマリ
- PGM 数: N 本（バッチ: X / CICS: Y）
- JCL ジョブ数: Z
- COPY ブック数: N（共有型定義の洗い出し完了）
- 移行困難項目: [REDEFINES複雑 / IMS / MQ Series / UTL_FILE 等]
- **COMP-3使用箇所**: 全PGMのCOMP-3フィールドを列挙（精度損失リスク）

### 2. PGMごとの移行設計

#### [DAILYRPT] → jobs/daily-report.ts

**元PGMの構造（Phase 1.5 より）:**
- WORKING-STORAGE: WS-AMOUNT PIC 9(9)V99 COMP-3
- PERFORM UNTIL EOF-FLAG: 入力ファイル1行ずつ処理
- CALL 'CALCUTIL': 金額計算
- EXEC SQL INSERT: 集計結果をDB書き込み

**移行後の設計:**
- `Decimal` 型 (decimal.js): WS-AMOUNT の精度を維持
- Node.js readline ストリーム: PERFORM UNTIL EOF 対応
- `calcUtil()` モジュール: CALCUTIL CALLの TypeScript 化
- Prisma INSERT: EXEC SQL の置き換え

**スプリント見積もり:** 型変換 0.5日 / バッチロジック 1日 / テスト 1日 = 計2.5日

### 3. 移行順序
1. COPY ブック → TypeScript 型定義（全PGMの前提）
2. COMP-3 変換ユーティリティ（全PGMの前提）
3. ユーティリティPGM（依存される側から先に）
4. バッチPGM（JCL実行順に従う）
5. CICS（最後）
```

---

## Phase 3: Sprint Planning — PDCAサイクル設計

**Phase 1.6 の移行設計書を唯一の入力とする。分析を活かさないまま実装に入ることを禁止する。**

### 3.0 事前共通タスク（Sprint 0 — 1回のみ）

```markdown
## Sprint 0: 基盤構築

- [ ] TypeScript + Node.js プロジェクト初期化
- [ ] COMP-3 変換ユーティリティ作成（decimal.js ベース）+ 単体テスト（精度検証）
- [ ] COPY ブック → TypeScript 型定義ファイル生成（`types/cobol.d.ts`）
- [ ] DB2 → PostgreSQL スキーマ移行（pgloader）
- [ ] EBCDIC → UTF-8 変換レイヤー（iconv-lite）
- [ ] CI/CD（GitHub Actions + Jest）
```

### 3.1 PGM 単位 PDCAサイクル（Sprint N ごとに繰り返す）

```markdown
## Sprint N: [PGM名] → [jobs/xxx.ts / modules/xxx.ts]

### 前提（Phase 1.6 より転記）
- 元PGMの責務: [1行定義]
- WORKING-STORAGE: [主要変数・COMP-3フィールド一覧]
- PERFORM 構造: [主要処理フロー]
- EXEC SQL: [使用クエリ一覧]
- CALL 先: [呼び出しモジュール]

### Step 1: 仕様書生成（TypeScript設計）
- [ ] WORKING-STORAGE → TypeScript 変数定義（COMP-3は全て Decimal 型）
- [ ] PERFORM UNTIL → async ループ設計
- [ ] EXEC SQL → Prisma / pg クエリ設計
- [ ] CALL → TypeScript モジュール import 設計
- 成果物: `docs/specs/[pgm-name].spec.md`

### Step 2: TypeScript 実装
- [ ] 型定義 + ビジネスロジック実装
- [ ] COMP-3 計算箇所は全て decimal.js で実装
- [ ] エラー処理: SQLCODE チェック → try/catch + ロールバック

### Step 3: テスト（精度検証が最重要）
- [ ] Jest 単体テスト（計算ロジック）
- [ ] COMP-3 精度テスト: 同じ入力で COBOL と TypeScript の出力が一致するか

### Step 4: 並行稼働テスト
- [ ] 本番 COBOL と Node.js に同じ入力を与えて出力比較
- 完了条件: 全出力が**ビット単位**で一致（金融計算の場合は特に）

### 振り返り
- COMP-3 変換で発生した精度差異:
- REDEFINES の解釈で詰まった箇所:
- 次 Sprint に流用できる変換パターン:
```

### 3.2 Sprint 計画一覧

```markdown
| Sprint | 対象 | 複雑度 | 実装 | テスト | 並行稼働 | 合計 |
|--------|------|--------|------|--------|---------|------|
| Sprint 0 | 基盤構築 | — | — | — | — | 1週 |
| Sprint 1 | COPY ブック型変換 | 低 | 1日 | 1日 | — | 2日 |
| Sprint 2 | [ユーティリティPGM] | 低 | 1日 | 1日 | 0.5日 | 2.5日 |
| Sprint N | [バッチPGM] | 中〜高 | 2日 | 2日 | 1日 | 5日 |
| Sprint X | [CICS PGM] | 高 | 3日 | 2日 | 1日 | 6日 |

**見積もりルール**: PGM数・行数が判明した時点で更新する。今の数字は仮置き。
```

---

## Phase 4: 重大リスク

| リスク | 影響 | 対策 |
|---|---|---|
| **COMP-3 精度損失** | 金融計算で誤差発生 | `decimal.js` / `big.js` 必須。`number` 型使用禁止 |
| **REDEFINES の複雑な解析** | データ構造の誤解釈 | Buffer レベルで解析。COBOL 仕様書と照合 |
| **文字コード（EBCDIC → UTF-8）** | 文字化け・データ破損 | iconv-lite で EBCDIC → UTF-8 変換レイヤー |
| **並行稼働期間のデータ整合** | 両システムで不整合 | 出力比較テストを自動化して差分ゼロを確認 |
| **IMS 階層DB** | リレーショナル設計が必要 | データモデルの根本再設計（別プロジェクト化を推奨） |

---

## 絶対禁止事項

- **金融計算で `number` 型使用禁止** — 必ず `decimal.js` / `big.js` を推奨として記載
- **COMP-3 の `parseInt` 変換禁止** — パック10進の正確なデコードが必要
- **AI voice禁止** / **途中停止禁止** / **工数根拠なし断言禁止**
