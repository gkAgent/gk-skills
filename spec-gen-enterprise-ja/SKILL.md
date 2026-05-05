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

<!-- merged from universal-spec-generator/common_rules.md on 2026-05-02 -->

---

## Appendix A: 生成開始ゲート（参照確認：必須）

### A.1 ゲートの目的
会話コンテキストの参照不安定により、誤前提で本文生成を開始する事故を防ぐ。

### A.2 ゲート手順
本文（テンプレート準拠の全文）生成前に、必ず **「参照できているコンテキスト要約（ゲート出力）」** を提示し、利用者の **OK** を得ること。OKが得られない場合、本文生成してはならない（質問フェーズへ）。

### A.3 ゲート出力に含める最低限（推測禁止）
UIハンドラ（Controller / PageModel / Router 等）を根拠に次を箇条書きで提示（取得不能は「取得不能」と明記）：
- クラス完全修飾名（FQCN / 言語に合わせた識別子）
- ハンドラメソッド一覧
- URL一覧（ルーティング宣言から取得）
- 遷移先ビュー名（戻り値 / レスポンスから取得できる範囲）
- 主なビュー受け渡し属性名
- 対象範囲（原則：UIハンドラクラス全体。例外時は Included / Excluded を明記）

### A.4 利用者への確認文（固定）
ゲート出力末尾に必ず記載：
- `この内容で対象を参照できています。詳細仕様書の本文生成に進めてよいですか？（OK/NG）`

### A.5 NG時の扱い
本文生成はせず、対象ソース・前版仕様書の追加・貼り付けを依頼する。

---

## Appendix B: MODE: CHANGE（改定管理）

### B.1 改定ID形式
- `YYYYMM:#<チケットNo>` に統一（例：`202602:#12345`）
- `YYYYMM` はリリース年月。チケット管理ツール（Redmine / Jira / GitHub Issues 等）のIDを `<チケットNo>` に使用。

### B.2 改定ID未提示時
- MODE: CHANGE なのに改定IDが無い場合：**MODE: NEW相当**として生成し、TODOに「改定ID未提示」を記載。

### B.3 前版仕様書が参照できない場合
前版がコンテキストに無い/参照できない場合：
- 改修後実装を正として **全文生成**（パッチ禁止）
- 「変更点サマリ」は作成しない（または「前版不在で作成不可」）
- 変更箇所マーキング（改定ID付与）は原則しない
- TODO に必ず記載：`MODE: CHANGE だが前版仕様書が参照できないため NEW 相当で生成した`

### B.4 前版参照できる場合の必須対応
- 0章「更新履歴」表に「改定ID」列を含める
- 章0直後に `0.x 変更点サマリ（改定ID: YYYYMM:#xxxxx）` を追加
- 変更箇所マーキング：
  - 原則：変更した見出し末尾に `【YYYYMM:#xxxxx】`
  - 見出しで表現不可：変更段落先頭に `(YYYYMM:#xxxxx)`

---

## Appendix C: 文字化け / エンコード運用

### C.1 想定エンコード
- **既定は UTF-8**。プロジェクト固有の事情で別エンコード（Shift_JIS / CP932 / EUC-JP 等）を使う場合は `[INPUT]` で明示する。

### C.2 取り扱い
- 文字化け疑い（`â€` 混入、`Ã`/`Â` 等の不自然な記号列）がある場合：
  1. 拡張子と想定エンコードを確認
  2. 利用者へ「想定エンコードで開き直して再提示」または「変換して再提示」を依頼
  3. 重要情報が壊れている箇所は質問/TODOに明記して保留（推測で復元しない）

---

## Appendix D: 図表の根拠境界（mermaid / 表）

- 図表は「根拠がある範囲」のみ作成。根拠のない遷移・処理・項目は描かず質問/TODOへ。
- 各図表の直前に必ず明記：
  - 根拠（例：UIハンドラのみ／UIハンドラ+ビュー／UIハンドラ+ユースケース層）
  - 未確定として省略した要素

### D.1 画面遷移図
- UIハンドラの戻り値 / リダイレクト / フォワード から読める遷移のみ記載。
- 根拠なしのリンク遷移は追加しない。

### D.2 シーケンス図
- UIハンドラ内で確認できる呼び出しのみ記載。
- DB更新・業務チェック等の内部処理は、呼び出しが確認できない限り追加しない。

---

## Appendix E: テンプレ準拠で必須の観点

以下の観点を最低限落とさない：
- 画面遷移図（mermaid）
- URL一覧（HTTPメソッド、ハンドラメソッド、戻り、用途）
- バリデーション（共通 / 業務）
- 二重送信防止 / 排他 / トランザクション
- 例外設計（ユーザ表示、ログレベル、復帰可否）
- DB設計（参照 / 更新テーブル、キー、SQL要点、インデックス観点）
- セキュリティ（権限、CSRF/XSRF、XSS、個人情報マスク）

---

## Appendix F: 生成後セルフチェック

生成直後に以下を確認し、結果を出力末尾に **「セルフチェック結果」** として明示する。

| チェック項目 | OK条件 | NG時の対応 |
|---|---|---|
| ハンドラメソッド網羅 | UIハンドラのメソッド一覧と仕様書のURL一覧が1対1で対応 | TODO「漏れの可能性」に具体メソッド名を列挙 |
| 推測断定ゼロ | 根拠のない断定記述が存在しない | 該当箇所をTODOに差し替え |
| テンプレ章立て | 全必須章（Appendix E）が存在する（省略時は理由を明記） | 不足章を追記 |
| 改定IDマーキング | MODE=CHANGE 時に変更箇所がマーキングされている | マーキング漏れを修正 |
| 文字化け疑いゼロ | 不自然な文字列が存在しない | Appendix C.2 の手順へ |

---

## Appendix G: TODO / Question 管理

### G.1 記載形式
```
<!-- TODO: [カテゴリ] 内容（どの章に影響するか） -->
```

カテゴリ一覧：
- `[ROOT]`：根拠不足（コンテキストに情報がない）
- `[SPEC]`：仕様未確定（利用者に確認が必要）
- `[DB]`：DB定義 / SQL要点が不明
- `[UI]`：ビュー / クライアントスクリプトの挙動が不明
- `[AUTH]`：権限 / 認証の仕様が不明
- `[FILE]`：ファイル名 / パスが確定できない

### G.2 TODO 一覧章
仕様書末尾の「TODO / 要確認事項」章に、全 TODO を以下の表でまとめる：

| No | カテゴリ | 内容 | 影響章 | 優先度 |
|---|---|---|---|---|
| 1 | [SPEC] | ○○の業務ルールが未確定 | 7.処理詳細 | 高 |
