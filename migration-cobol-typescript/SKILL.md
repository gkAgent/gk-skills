---
name: migration-cobol-typescript
description: Guides COBOL legacy system migration to TypeScript/Node.js with a structured methodology covering mainframe data structures, batch processing, and DB2 migration. Use when modernizing COBOL batch programs or CICS online transactions to TypeScript microservices or Node.js batch jobs, assessing COBOL migration feasibility, converting COBOL data structures (COMP-3, OCCURS, REDEFINES) to TypeScript types, or planning mainframe decommission. Includes DB2 to PostgreSQL migration, JCL to Node.js/Docker job conversion, and COBOL PICTURE clause to TypeScript type mapping.
---

# Migration — COBOL → TypeScript / Node.js

> The hardest migration. But someone has to do it.
> For COBOL → DB2 spec documentation, see **spec-gen-cobol-db2**.

---

## Phase 0: Scope

Ask the user:

> 移行対象を教えてください。
>
> **現行**: COBOL ランタイム / 実行環境（JCLバッチ / CICS / IMS）
> **移行先**: TypeScript + Node.js バッチ / NestJS マイクロサービス / Next.js API
> **DB**: DB2 for z/OS / DB2 LUW → PostgreSQL / MySQL / SQL Server
> **規模**: PGM数 / COPY ブック数 / JCLジョブ数

---

## Phase 1: Assessment

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
- COBOL の10進固定小数点演算（金融計算） → `decimal.js` / `big.js` で精度保証
- `SORT` / `MERGE` ステートメント → DB ORDER BY またはメモリソート

---

## Phase 2: Conversion Patterns

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

## Phase 3: Migration Task List

```markdown
### フェーズ A: データ層
- [ ] COPY ブック → TypeScript 型定義マッピング表
- [ ] DB2 → PostgreSQL スキーマ移行（pgloader / ora2pg相当）
- [ ] COMP-3 / 固定小数点 → decimal.js 変換ユーティリティ作成

### フェーズ B: ビジネスロジック
- [ ] [PGM名] バッチロジック → Node.js 関数変換
- [ ] EXEC SQL → Prisma/pg クエリ変換
- [ ] CALL 外部プログラム → TypeScript モジュール化

### フェーズ C: 実行基盤
- [ ] JCL ジョブ → node-cron / BullMQ スケジューラ
- [ ] エラー処理: SQLCODE チェック → try/catch + ロールバック
- [ ] ファイル I/O: VSAM → PostgreSQL / S3 移行判断

### フェーズ D: CICS（該当時）
- [ ] BMS マップ → React/HTML フォーム再設計
- [ ] EXEC CICS LINK → REST API 呼び出し
- [ ] トランザクション境界 → DB トランザクション

### フェーズ E: テスト・並行稼働
- [ ] Jest 単体テスト（計算ロジック精度検証）
- [ ] 本番 COBOL と Node.js の出力比較（並行稼働期間）
- [ ] 金融計算の精度検証（小数点以下 N 桁の一致確認）
```

---

## Phase 4: Critical Risks

| リスク | 影響 | 対策 |
|---|---|---|
| **COMP-3 精度損失** | 金融計算で誤差発生 | `decimal.js` / `big.js` 必須。`number` 型使用禁止 |
| **REDEFINES の複雑な解析** | データ構造の誤解釈 | Buffer レベルで解析。COBOL 仕様書と照合 |
| **文字コード（EBCDIC → UTF-8）** | 文字化け・データ破損 | iconv-lite で EBCDIC → UTF-8 変換レイヤー |
| **並行稼働期間のデータ整合** | 両システムで不整合 | 出力比較テストを自動化して差分ゼロを確認 |
| **IMS 階層DB** | リレーショナル設計が必要 | データモデルの根本再設計（別プロジェクト化を推奨） |

---

## Forbidden

- **金融計算で `number` 型使用禁止** — 必ず `decimal.js` / `big.js` を推奨として記載
- **COMP-3 の `parseInt` 変換禁止** — パック10進の正確なデコードが必要
- **AI voice禁止** / **途中停止禁止** / **工数根拠なし断言禁止**
