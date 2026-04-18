---
name: code-review-cobol
description: Structured code review for COBOL programs (IBM Enterprise COBOL, Micro Focus, GnuCOBOL) with DB2 and VSAM. Use when reviewing COBOL batch programs before production release, auditing legacy COBOL code for modernization assessment, or checking COBOL quality in a mainframe migration project. Extends code-review-enterprise-ja. Covers SQLCODE handling completeness, CURSOR lifecycle symmetry, WORKING-STORAGE sizing vs DB column definitions, CICS transaction boundaries, and common COBOL anti-patterns.
---

# Code Review — COBOL × DB2 / VSAM

> Full coverage across 5 languages: install **code-review-enterprise-ja**.
> This skill focuses on COBOL.

---

## Activation

Ask: レビュー対象の PGM名 と実行環境（JCLバッチ / CICS / IMS）を教えてください。

---

## COBOL Review Checklist

### A. データ定義

- `WORKING-STORAGE` の `PIC` サイズが DB2 カラム定義と一致しているか
- `COMP-3`（パック10進）の桁数とバイト長の整合: `PIC 9(n)` → `CEIL((n+1)/2)` バイト
- `COMP` / `COMP-4` の桁数（ハーフワード: 1-4桁、フルワード: 5-9桁、ダブルワード: 10-18桁）
- 文字列変数のトリミング: `MOVE SPACE TO` 初期化が必要な箇所

### B. SQL（EXEC SQL）

- 全 `EXEC SQL` の後に `SQLCODE` チェックがあるか（`IF SQLCODE = 0` or `EVALUATE SQLCODE`）
- `SQLCODE +100`（NOT FOUND）の明示的ハンドリング
- `SQLCODE -803`（重複）/ `-911`（デッドロック）の処理
- `CURSOR`の `OPEN` / `FETCH` / `CLOSE` が対称になっているか
- `WITH HOLD` カーソルの COMMIT 後の扱い
- `FOR UPDATE OF` 使用時の `WHERE CURRENT OF` の正確性

### C. ファイル操作（VSAM / 順編成）

- `OPEN` 後の `FILE STATUS` 確認（`IF FS-CODE NOT = '00'`）
- `CLOSE` 漏れ（`GOTO` / `PERFORM THRU` による制御フロー中断時）
- `READ INTO` / `WRITE FROM` のバッファ初期化

### D. 構造・品質

- `GOTO` 文の使用（スパゲッティ化リスク — モダナイゼーション阻害要因として記録）
- `ALTER` 文の使用（保守不可能 — 即記録）
- `ON ERROR CONTINUE` 等の例外飲み込みパターン
- `COPY` ブックのバージョン整合性

### E. CICS固有（該当時）

- `EXEC CICS HANDLE ABEND` の適切な使用
- `SYNCPOINT` の位置（トランザクション境界が適切か）
- `EXEC CICS RETURN` のロジック
- `COMMAREA` のサイズ制限（32,767バイト）

---

## Review Report Output

```markdown
## レビュー結果: [PGM名]

### 重大（即修正）
- [段落名] EXEC SQL FETCH後にSQLCODEチェックなし。-811（複数行）が無視される

### 警告（修正推奨）
- [段落名] CURSORのCLOSEが1ルートで欠落。リソースリーク

### 情報
- GOTO文が3箇所。モダナイゼーション時の優先リファクタリング候補

### 総評
[断定口調で1〜3行。モダナイゼーション判定を含める]
```

---

## Forbidden

- **SQLCODE チェック漏れの見落とし禁止** — 全 EXEC SQL を確認する
- **GOTO の無視禁止** — 存在を必ず記録する（レガシー資産の正確な現状把握）
- **途中停止禁止** / **AI voice禁止**
