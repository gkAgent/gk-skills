---
name: spec-gen-enterprise-ja
description: Generates structured design specification documents (詳細仕様書) from existing source code for Japanese enterprise systems. Supports Java × Oracle/PostgreSQL/MySQL/SQL Server, TypeScript × all DBs, C#/.NET × all DBs, VB.NET × all DBs, COBOL × all DBs. Use when a user wants to generate a spec document from legacy source code, document an undocumented system, create 詳細仕様書 from existing implementation, or produce design documents for a maintenance project. Follows the 5 D-principles: question-driven start, coverage visualization, Files>Prompts, 80%-NOT instructions, and no AI-voice filler.
---

# Spec Generation Kit — Enterprise Japanese Edition

## Activation

When this skill is invoked, follow the exact flow below. Do not skip phases.

---

## Phase 0: Identify Language × DB

Ask the user:

> このシステムの言語と DB の組み合わせを教えてください。
>
> **言語**: Java / TypeScript / C# (.NET) / VB.NET / COBOL / その他
> **DB**: Oracle / PostgreSQL / MySQL / SQL Server / SQLite / DBなし / その他

If the user has already provided this context, skip to Phase 1.

> 粒度と実行モードを選んでください。
>
> **Granularity（粒度）**:
> - `Project` — VB.NET/C# WinForms/WebForms 推奨（プロジェクト全体を1ファイルに）
> - `Feature` — Spring Boot / ASP.NET Core など機能単位が自然なシステム向け
> - `Form` — フォーム単位の細粒度（UI変更詳細が必要な時）
> - `Module` — バッチ、サービス層など中粒度
>
> **ExecutionMode**:
> - `Review` — 1仕様書ずつ生成→確認→次へ（品質重視・初回利用向け）
> - `Batch` — 途中確認なしで全対象を一気に完了まで実行（100PJ一晩処理向け）

---

## Phase 1: Question-Driven Setup (D1)

**Never start generating the spec without first completing this phase.**

Ask in three stages. Wait for answers before proceeding.

### Stage 1 — System basics
1. システム名（正式名称）は？
2. 対象の機能・画面・バッチの名前は？（複数可）
3. ソースコード・DDL・設定ファイルは添付済みですか？

### Stage 2 — Tech stack (language-specific)

**Java users:**
- Java バージョン（8 / 11 / 17 / 21）
- フレームワーク（Spring Boot / Spring MVC / Jakarta EE / 純粋 Java）
- データアクセス方式（JPA/Hibernate / MyBatis / JDBC Template / jOOQ / PL/SQL呼び出し）
- Oracle の場合: バージョン、PL/SQL Package の有無

**TypeScript users:**
- フレームワーク（React / Next.js / Vue / Angular / Express / NestJS）
- ORM/クライアント（Prisma / TypeORM / Drizzle / Knex / 生SQL）

**C# / VB.NET users:**
- .NET バージョン
- プロジェクト種別（WinForms / WPF / ASP.NET Core MVC / Web API / WebForms）
- データアクセス（Entity Framework Core / Dapper / ADO.NET / ストアドプロシージャ）

**COBOL users:**
- ランタイム（IBM Enterprise COBOL / Micro Focus / OpenCOBOL）
- DB アクセス方式（EXEC SQL / VSAM / DB2 / IMS）

### Stage 3 — Output preferences
- 粒度：`Project` / `Feature` / `Form` / `Module`（Phase 0で決定済みの場合はスキップ）
- 実行モード：`Review`（確認しながら）/ `Batch`（一気に最後まで）
- 出力形式：Markdown / 既定テンプレートに沿って
- 優先度が高い機能はありますか？

After collecting answers, **construct the `[INPUT]` block and show it to the user for confirmation** before generating.

---

## Phase 2: Coverage Plan (D2)

Before writing any spec, output a **踏破計画 (Coverage Plan)**:

```
## 踏破計画

| # | 対象 | 種別 | ステータス |
|---|------|------|-----------|
| 1 | [機能名] | 画面/バッチ/API | [ ] 未着手 |
| 2 | ... | ... | [ ] 未着手 |
```

Rules:
- List ALL targets identified from the provided source code or user input
- Do NOT choose which ones to process based on your own judgment
- After each item, update the status to `[x] 完了` or `[△] TODO あり`
- Never stop processing mid-list unless the user explicitly says so

**Batch モードの場合**: 踏破計画を表示後、確認を待たずに全項目を順次処理する。処理完了後にサマリーを出力する。
**Review モードの場合**: 各項目完了ごとに確認を求める（デフォルト）。

---

## Phase 3: Spec Generation (per item)

For each item in the coverage plan, generate a spec following this structure:

```markdown
# 詳細仕様書 — [機能名]

## 0. 基本情報
| 項目 | 内容 |
|---|---|
| システム名 | |
| 機能名 | |
| 対象ファイル | |
| 言語/FW/DB | |
| 作成日 | |

## 1. 処理概要
[1〜3行で端的に]

## 2. 入力
### 2.1 パラメータ / 画面入力
| 項目名 | 型 | 必須 | 説明 |
|---|---|---|---|

## 3. 処理フロー
[番号付きステップ。根拠があるものだけ記述]

## 4. DB 操作
| 種別 | テーブル/View | 条件 | 備考 |
|---|---|---|---|

## 5. 出力 / 戻り値

## 6. エラー処理

## 7. TODO / 要確認事項
[根拠不足の事項をすべてここに列挙。空にしてはいけない場合は正直にリストアップ]
```

### 出力ファイル命名規則

| Granularity | ファイル名 |
|---|---|
| Project | `spec_<projectname>.md` |
| Feature | `spec_<featurename>.md` |
| Form | `spec_<projectname>_<formname>.md` |
| Module | `spec_<modulename>.md` |

---

## Phase 4: TODO Resolution（2パス目）

After generating specs, if any `TODO` items remain:

> 仕様書内に TODO が残っています。`todo-resolution-ja` スキルで一括解消できます。
>
> 「todo解消」と入力すると Phase 2（2パス）を開始します。

**Batch モードの場合**: 全仕様書生成完了後に自動的に TODO 一覧を集計して提示する。

**Review モードの場合**: 各仕様書生成後に TODO があれば即時提示する。

---

## Forbidden — 絶対禁止事項 (D4)

These rules take priority over everything else:

- **推測断定禁止**: ソースに根拠がない事項を断定として記述しない。「〜と思われます」「一般的に〜」も禁止
- **途中停止禁止**: 踏破計画を完了させる前に止まらない
- **AI voice 禁止 (D5)**: 「本システムは〜」「〜することが重要です」などの冗長な導入文を書かない
- **100文字超プロンプト要求禁止 (D3)**: 利用者に長いプロンプトを書かせない
- **根拠なしTODO削除禁止**: 不明事項は TODO に書く。TODO を省いて見た目をよくしない

---

## Notes for specific language × DB combinations

### Java × Oracle
- PL/SQL Package 呼び出しがある場合は `4. DB操作` に Package 名・Procedure 名・IN/OUT引数を記載
- `SYS_REFCURSOR` の扱いは必ず明記
- Oracle バージョン固有機能（Partitioning / Advanced Queuing 等）は根拠があるときのみ記載

### TypeScript × any DB
- `async/await` エラーハンドリングパターンを明記
- Prisma の場合は schema.prisma のモデル定義を参照元として明示

### VB.NET / C# × SQL Server
- ストアドプロシージャ呼び出しは `CommandType.StoredProcedure` の使用有無を明記
- トランザクション管理（`SqlTransaction` / `TransactionScope`）を記載

### COBOL × DB2
- EXEC SQL ブロックごとに SQLCODE チェックを記載
- WORKING-STORAGE の HOST 変数一覧を仕様書に含める
