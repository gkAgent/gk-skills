---
name: spec-gen-cobol-oracle
description: Generates detailed design specs (詳細仕様書) for COBOL × Oracle systems using Pro*COBOL or embedded SQL. Use when documenting COBOL batch programs or online transactions backed by Oracle, reverse-engineering undocumented COBOL/Oracle systems, or producing 詳細仕様書 for COBOL × Oracle modernization projects. Covers EXEC SQL blocks, SQLCA / SQLCODE handling, CURSOR lifecycle, host variable declarations, Pro*COBOL precompile flow, and PL/SQL stored procedure calls. Extends spec-gen-enterprise-ja.
---

# Spec Generation — COBOL × Oracle

> COBOL から Oracle に Pro*COBOL（プリコンパイラ）経由でアクセスする案件向け。
> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。

---

## Phase 0: Scope

Ask the user:

> COBOL × Oracle のプロジェクトを教えてください。
>
> - **COBOL ランタイム**: IBM Enterprise COBOL / Micro Focus / GnuCOBOL / 富士通 NetCOBOL
> - **Pro*COBOL バージョン**: Oracle Database 11g / 19c に同梱されたもの
> - **実行環境**: バッチ（JCL / シェル） / オンライン（CICS / 独自TPモニタ）
> - **ソース**: `.pco` / `.cbl` / コピーブック（`.cpy`）を添付できますか？

---

## Phase 1: Question-Driven Setup

**Stage 1 — Basics**
1. プログラム名（PGM名）と処理概要は？
2. WORKING-STORAGE のホスト変数定義は添付できますか？
3. EXEC SQL 文は何件ありますか（おおよそ）？

**Stage 2 — COBOL specifics**
- `IDENTIFICATION DIVISION` の PROGRAM-ID は？
- コピーブック（`COPY 〜`）の参照状況
- `PERFORM` / `CALL` による外部プログラム呼び出しの有無
- バッチであれば JCL / シェルからの呼び出しパラメータ

**Stage 3 — Oracle specifics**
- `EXEC SQL INCLUDE SQLCA` の有無
- カーソル（`DECLARE CURSOR`）の数
- `FOR UPDATE` / `WHERE CURRENT OF` の使用有無
- PL/SQL ストアド（パッケージ / プロシージャ / ファンクション）の呼び出し有無

---

## Phase 2: Coverage Plan

EXEC SQL ブロックを1件ずつリストアップしてから処理開始。

| # | 箇所（段落名/行番号） | SQL種別 | SQLCODE処理 | ステータス |
|---|---|---|---|---|
| 1 | 1000-INIT-PROCESS | SELECT | ✅ | [ ] |

---

## Oracle 固有パターン（COBOL 視点）

### SQLCA インクルードと SQLCODE 分岐

```cobol
       WORKING-STORAGE SECTION.
           EXEC SQL INCLUDE SQLCA END-EXEC.

       PROCEDURE DIVISION.
           EXEC SQL
               SELECT EMP_NAME
                 INTO :WS-EMP-NAME
                 FROM EMPLOYEE
                WHERE EMP_ID = :WS-EMP-ID
           END-EXEC.

           EVALUATE SQLCODE
               WHEN 0
                   PERFORM 2000-FOUND
               WHEN 1403
                   PERFORM 2100-NOT-FOUND
               WHEN OTHER
                   PERFORM 9000-DB-ERROR
           END-EVALUATE.
```

仕様書に記載すること:
- 各 EXEC SQL の後に SQLCODE 分岐があるか
- 1403（NOT FOUND）の処理内容
- マイナス値（-1, -1017 等）の例外ハンドリング設計

### NUMBER 型 → COBOL ホスト変数

```cobol
       01  WS-EMP-ID        PIC 9(06) COMP-3.       *> NUMBER(6)
       01  WS-SALARY        PIC S9(8)V9(2) COMP-3.  *> NUMBER(10,2)
       01  WS-EMP-NAME      PIC X(40).              *> VARCHAR2(40)
       01  WS-HIRE-DATE     PIC X(10).              *> DATE（書式は明示記載）
```

仕様書に記載すること:
- NUMBER 精度と PIC 句の対応表
- DATE 型を文字列で受ける場合の書式マスク（`YYYY-MM-DD HH24:MI:SS` 等）

### カーソル定義と OPEN/FETCH/CLOSE 対称性

```cobol
           EXEC SQL
               DECLARE CSR-EMP CURSOR FOR
                   SELECT EMP_ID, EMP_NAME, SALARY
                     FROM EMPLOYEE
                    WHERE DEPT_ID = :WS-DEPT-ID
                    ORDER BY EMP_ID
           END-EXEC.

           EXEC SQL OPEN CSR-EMP END-EXEC.
           PERFORM UNTIL SQLCODE NOT = 0
               EXEC SQL
                   FETCH CSR-EMP INTO :WS-EMP-ID, :WS-EMP-NAME, :WS-SALARY
               END-EXEC
           END-PERFORM.
           EXEC SQL CLOSE CSR-EMP END-EXEC.
```

### PL/SQL ストアド呼び出し

```cobol
           EXEC SQL
               CALL PKG_EMPLOYEE.UPDATE_SALARY(:WS-EMP-ID, :WS-NEW-SALARY)
           END-EXEC.
```

---

## COBOL × Oracle 仕様書テンプレート

```markdown
# 詳細仕様書 — [プログラム名]

## 0. 基本情報
| 項目 | 内容 |
|---|---|
| PGM名 | XXXXXXXX |
| ランタイム | IBM Enterprise COBOL |
| Oracle | 19c |
| Pro*COBOL | 19.x |
| 実行環境 | JCL バッチ |

## 1. 処理概要

## 2. WORKING-STORAGE ホスト変数一覧
| 変数名 | PIC | 対応カラム | Oracle 型 | 備考 |
|---|---|---|---|---|
| WS-EMP-ID | PIC 9(6) COMP-3 | EMPLOYEE.EMP_ID | NUMBER(6) | |
| WS-SALARY | PIC S9(8)V9(2) COMP-3 | EMPLOYEE.SALARY | NUMBER(10,2) | |

## 3. EXEC SQL 操作一覧
| # | 段落 | 種別 | テーブル/PKG | 条件 | SQLCODE処理 |
|---|---|---|---|---|---|
| 1 | 1000-SELECT | SELECT | EMPLOYEE | EMP_ID = :WS-EMP-ID | 0/1403/その他 |

## 4. カーソル定義
| カーソル名 | 対象テーブル | OPEN/FETCH/CLOSE | FOR UPDATE |
|---|---|---|---|

## 5. SQLCODE 設計
| SQLCODE | 意味 | 処理内容 |
|---|---|---|
| 0 | 正常 | 続行 |
| +1403 | NOT FOUND | 〇〇フラグON |
| -1 | 一意制約違反 | ロールバック |
| -1017 | 認証エラー | プロセス停止 |

## 6. TODO / 要確認事項
```

---

## Forbidden

- **SQLCODE チェック漏れの見落とし禁止**: すべての EXEC SQL の後に分岐があるか確認、無ければ「TODO: SQLCODE チェック未実装」と明記
- **カーソル CLOSE 漏れ禁止**: OPEN/CLOSE の対称性を必ず確認、非対称なら「TODO: リソースリーク可能性」
- **NUMBER 精度の省略禁止**: `NUMBER` だけでなく `NUMBER(precision, scale)` を必ず仕様書に転記
- **DATE 書式の省略禁止**: 文字列で受ける場合の書式マスクを必ず明記
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
