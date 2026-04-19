# Spec Gen — COBOL × DB2

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-cobol-db2` で自動起動します（貼り付け不要）。

---

以下の指示に従って、COBOL × DB2 システムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

> 全言語・全DBの網羅版は **spec-gen-enterprise-ja** を使うこと。
> このスキルは COBOL × IBM DB2（z/OS および LUW）に特化しています。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> COBOL × DB2 のプロジェクトを教えてください。
>
> - **COBOL ランタイム**: IBM Enterprise COBOL / Micro Focus / GnuCOBOL
> - **DB2 種別**: DB2 for z/OS / DB2 LUW / IBM Db2 on Cloud
> - **実行環境**: バッチ（JCL） / オンライン（CICS） / IMS
> - **ソース**: `.cbl` / `.cob` / コピーブック（`.cpy`）を添付できますか？

---

## Phase 1: 質問フェーズ

**Stage 1 — 基本情報**
1. プログラム名（PGM名）と処理概要は？
2. WORKING-STORAGE のホスト変数定義は添付できますか？
3. EXEC SQL 文は何件ありますか（おおよそ）？

**Stage 2 — COBOL 固有情報**
- `IDENTIFICATION DIVISION` の PROGRAM-ID は？
- コピーブック（`COPY 〜`）の参照状況
- `PERFORM` / `CALL` による外部プログラム呼び出しの有無
- CICS の場合: `EXEC CICS RECEIVE` / `SEND` / `SYNCPOINT` の使用有無

**Stage 3 — DB2 固有情報**
- `SQLCA` or `SQL INCLUDE SQLCA` の使用
- カーソル（`DECLARE CURSOR`）の数
- `FOR UPDATE OF` / `WITH HOLD` の使用有無
- `CALL`ステートメントによるストアドプロシージャ呼び出しの有無

---

## Phase 2: カバレッジプラン

EXEC SQL ブロックを1件ずつリストアップしてから処理を開始してください。

| # | 箇所（段落名/行番号） | SQL種別 | SQLCODE処理 | ステータス |
|---|---|---|---|---|
| 1 | 1000-INIT-PROCESS | SELECT | ✅ | [ ] |

---

## Phase 3: 仕様書生成 — COBOL × DB2 テンプレート

```markdown
# 詳細仕様書 — [プログラム名]

## 0. 基本情報
| 項目 | 内容 |
|---|---|
| PGM名 | XXXXXXXX |
| ランタイム | IBM Enterprise COBOL |
| DB2 | z/OS V13 |
| 実行環境 | JCL バッチ |

## 1. 処理概要

## 2. WORKING-STORAGE ホスト変数一覧
| 変数名 | PIC | 対応カラム | 備考 |
|---|---|---|---|
| WS-EMP-ID | PIC 9(6) | EMP.EMP_ID | COMP-3 |

## 3. EXEC SQL 操作一覧
| # | 段落 | 種別 | テーブル | 条件 | SQLCODE処理 |
|---|---|---|---|---|---|
| 1 | 1000-SELECT | SELECT | EMP | EMP_ID = :WS-EMP-ID | +100: NOT FOUND分岐 |

## 4. カーソル定義
| カーソル名 | 対象テーブル | OPEN/FETCH/CLOSE | WITH HOLD |
|---|---|---|---|

## 5. エラー処理
| SQLCODE | 意味 | 処理内容 |
|---|---|---|
| 0 | 正常 | 続行 |
| +100 | NOT FOUND | 〇〇フラグON |
| -803 | 重複 | ロールバック |

## 6. TODO / 要確認事項
```

---

## 絶対禁止事項

- **SQLCODE チェック漏れの見落とし禁止**: すべての EXEC SQL ブロックの後に SQLCODE 確認があるか記載する。ない場合は「TODO: SQLCODE チェック未実装」と明記
- **カーソル CLOSE 漏れの見落とし禁止**: OPEN と CLOSE が対称になっているか確認し、非対称なら「TODO: リソースリーク可能性」と記載
- **CICS SYNCPOINT の確認必須**: CICS環境ではトランザクション境界を明記する
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
