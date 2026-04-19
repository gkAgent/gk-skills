# 仕様書生成キット — エンタープライズ日本語版

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/spec-gen-enterprise-ja` で自動起動します（貼り付け不要）。

---

以下の指示に従って、日本語エンタープライズシステムの詳細仕様書を生成してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: 言語・DB・粒度・実行モードの確認

まず以下を質問してください:

> このシステムの言語と DB の組み合わせを教えてください。
>
> **言語**: Java / TypeScript / C# (.NET) / VB.NET / COBOL / その他
> **DB**: Oracle / PostgreSQL / MySQL / SQL Server / SQLite / DBなし / その他

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

すでにコンテキストが提供されている場合は Phase 1 に進んでください。

---

## Phase 1: 質問駆動セットアップ

**このフェーズを完了する前に仕様書生成を始めないこと。**

3段階で質問し、回答を待ってから次に進んでください。

### Stage 1 — システム基本情報
1. システム名（正式名称）は？
2. 対象の機能・画面・バッチの名前は？（複数可）
3. ソースコード・DDL・設定ファイルは添付済みですか？

### Stage 2 — 技術スタック（言語別）

**Java の場合:**
- Java バージョン（8 / 11 / 17 / 21）
- フレームワーク（Spring Boot / Spring MVC / Jakarta EE / 純粋 Java）
- データアクセス方式（JPA/Hibernate / MyBatis / JDBC Template / jOOQ / PL/SQL呼び出し）
- Oracle の場合: バージョン、PL/SQL Package の有無

**TypeScript の場合:**
- フレームワーク（React / Next.js / Vue / Angular / Express / NestJS）
- ORM/クライアント（Prisma / TypeORM / Drizzle / Knex / 生SQL）

**C# / VB.NET の場合:**
- .NET バージョン
- プロジェクト種別（WinForms / WPF / ASP.NET Core MVC / Web API / WebForms）
- データアクセス（Entity Framework Core / Dapper / ADO.NET / ストアドプロシージャ）

**COBOL の場合:**
- ランタイム（IBM Enterprise COBOL / Micro Focus / OpenCOBOL）
- DB アクセス方式（EXEC SQL / VSAM / DB2 / IMS）

### Stage 3 — 出力設定
- 粒度：`Project` / `Feature` / `Form` / `Module`（Phase 0で決定済みの場合はスキップ）
- 実行モード：`Review`（確認しながら）/ `Batch`（一気に最後まで）
- 出力形式：Markdown / 既定テンプレートに沿って
- 優先度が高い機能はありますか？

回答を収集したら、**`[INPUT]` ブロックを構築してユーザーに確認を求めてから**生成を開始してください。

---

## Phase 2: 踏破計画

仕様書生成前に必ず **踏破計画** を出力してください:

```
## 踏破計画

| # | 対象 | 種別 | ステータス |
|---|------|------|-----------|
| 1 | [機能名] | 画面/バッチ/API | [ ] 未着手 |
| 2 | ... | ... | [ ] 未着手 |
```

ルール:
- 提供されたソースコードまたはユーザー入力から特定できる **全対象** をリストアップ
- 自分の判断で処理する項目を選ばない
- 各項目完了後にステータスを `[x] 完了` または `[△] TODO あり` に更新する
- ユーザーが明示的に指示しない限り途中で止まらない

**Batch モードの場合**: 踏破計画を表示後、確認を待たずに全項目を順次処理する。処理完了後にサマリーを出力する。
**Review モードの場合**: 各項目完了ごとに確認を求める（デフォルト）。

---

## Phase 3: 仕様書生成（項目ごと）

踏破計画の各項目に対して、以下の構造で仕様書を生成してください:

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

## Phase 4: TODO 解消（2パス目）

仕様書生成後に TODO が残っている場合:

> 仕様書内に TODO が残っています。続けて TODO を一括解消しますか？
> 「はい」と答えると、全 TODO を抽出→分類→質問リスト生成の順で進めます。

**Batch モードの場合**: 全仕様書生成完了後に自動的に TODO 一覧を集計して提示する。
**Review モードの場合**: 各仕様書生成後に TODO があれば即時提示する。

TODO 解消フロー:
1. 全 TODO を抽出してカテゴリ別（DB系 / 業務ルール系 / UI系 / 非機能系）に分類
2. カテゴリごとに具体的な質問リストを生成
3. 回答を受けて仕様書を差分更新

---

## 絶対禁止事項

- **推測断定禁止**: ソースに根拠がない事項を断定として記述しない。「〜と思われます」「一般的に〜」も禁止
- **途中停止禁止**: 踏破計画を完了させる前に止まらない
- **AI冗長文禁止**: 「本システムは〜」「〜することが重要です」などの導入文を書かない
- **長いプロンプト要求禁止**: ユーザーに100文字超のプロンプトを書かせない
- **TODO削除禁止**: 不明事項は TODO に書く。省いて見た目をよくしない

---

## 言語×DB 固有の注意事項

### Java × Oracle
- PL/SQL Package 呼び出しがある場合は §4 に Package名・Procedure名・IN/OUT引数を記載
- `SYS_REFCURSOR` の扱いは必ず明記

### TypeScript × 任意DB
- `async/await` エラーハンドリングパターンを明記
- Prisma の場合は schema.prisma のモデル定義を参照元として明示

### VB.NET / C# × SQL Server
- ストアドプロシージャ呼び出しは `CommandType.StoredProcedure` の使用有無を明記
- トランザクション管理（`SqlTransaction` / `TransactionScope`）を記載

### COBOL × DB2
- EXEC SQL ブロックごとに SQLCODE チェックを記載
- WORKING-STORAGE の HOST 変数一覧を仕様書に含める
