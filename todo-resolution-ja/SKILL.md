---
name: todo-resolution-ja
description: Resolves TODO items in design specification documents through a structured 2-pass methodology. Use when a spec document or design document has accumulated TODO comments that need resolution, when a user wants to systematically clear outstanding questions in a specification, when preparing a spec for final review by removing open items, or when transitioning a draft spec to approved status. Works with any language or domain. Generates targeted questions for each TODO, collects answers, and produces a clean diff-based update.
---

# TODO Resolution Kit — 2-Pass Methodology

## Activation

When this skill is invoked, follow the 2-pass flow: **Extract → Categorize → Question → Update**.

---

## Pass 1: Extract & Categorize

### Step 1.1: Extract all TODOs

Scan the provided document(s) and list every TODO item:

```markdown
## TODO 抽出リスト

| # | 場所（セクション名 / 行） | TODO 内容 | カテゴリ |
|---|---|---|---|
| T-001 | §4 DB操作 | 更新対象テーブルを確認 | DB系 |
| T-002 | §2 入力 | バリデーションルール不明 | 業務ルール系 |
| T-003 | §5 出力 | エラーメッセージ文言未定 | UI系 |
```

Do not skip any TODO. Even "後で確認" style TODOs must be listed.

### Step 1.2: Categorize

Assign each TODO to one of these categories:

| カテゴリ | 判断者 | 例 |
|---|---|---|
| **DB系** | DBA / 開発者 | テーブル名・カラム名・SQL条件・インデックス |
| **業務ルール系** | 業務担当者 / PO | バリデーション・承認フロー・権限・状態遷移 |
| **UI系** | デザイナー / PO | 画面文言・表示条件・レイアウト・エラーメッセージ |
| **非機能系** | アーキテクト | 性能目標・タイムアウト値・同時接続数 |
| **外部IF系** | 連携先担当 | API仕様・ファイル形式・通信プロトコル |
| **調査必要** | 開発者 | ソースコード確認・既存実装の動作確認 |

### Step 1.3: Priority assignment

Mark each TODO with priority:

- 🔴 **P1 ブロッカー**: この情報なしでは実装できない
- 🟡 **P2 実装前**: 実装開始前に解決が望ましい
- 🟢 **P3 後回し可**: リリース後でも影響が小さい

---

## Pass 2: Question Generation & Document Update

### Step 2.1: Generate questions

For each TODO, generate a specific, answerable question:

```markdown
## 確認事項リスト

### 🔴 P1 ブロッカー

**[T-001] DB系 — §4 DB操作**
> 質問: ユーザー登録処理で更新されるテーブルを教えてください。
> 選択肢: (A) users テーブルのみ (B) users + user_profiles (C) その他
> 補足: ソースコードの [ファイル名:行番号] に INSERT が見えますが、テーブル名が変数になっており確認できませんでした。

**[T-002] 業務ルール系 — §2 入力**
> 質問: メールアドレスの重複チェックはどのタイミングで行いますか？
> 選択肢: (A) 入力中リアルタイム (B) 送信ボタン押下時 (C) なし（DB制約のみ）
> 補足: 既存コードに重複チェックのロジックが見当たりませんでした。

### 🟡 P2 実装前

**[T-003] UI系 — §5 出力**
> 質問: バリデーションエラー時のメッセージ文言を決定してください。
> 例: 「メールアドレスの形式が正しくありません」
> 補足: 他画面のエラーメッセージ形式があれば統一して使用します。
```

### Step 2.2: Collect answers

Present all questions grouped by category. Wait for the user's answers before proceeding to Step 2.3.

If the user provides partial answers, acknowledge what was answered and ask about the remaining items specifically.

### Step 2.3: Diff update

After receiving answers, produce a **diff-format update** for each resolved TODO:

```markdown
## ドキュメント更新差分

### T-001 解決: §4 DB操作

**変更前:**
> 更新対象テーブルを確認 <!-- TODO -->

**変更後:**
> | 種別 | テーブル | 条件 | 備考 |
> |---|---|---|---|
> | INSERT | users | - | ユーザー登録 |
> | INSERT | user_profiles | users.id FK | プロフィール初期化 |

### T-002 解決: §2 入力

**変更前:**
> バリデーションルール不明 <!-- TODO -->

**変更後:**
> | 項目 | ルール |
> |---|---|
> | メールアドレス | RFC5322形式、DB重複チェック（送信時） |
```

### Step 2.4: Remaining TODO summary

After applying all resolved TODOs, output:

```markdown
## 残存 TODO サマリー

| # | TODO内容 | カテゴリ | 理由 | 次のアクション |
|---|---|---|---|---|
| T-XXX | [内容] | [カテゴリ] | [未解決の理由] | [誰に何を確認するか] |

未解決 TODO 数: X件 / 全体: Y件
解決率: Z%
```

---

## Rules

**Do not remove a TODO without a confirmed answer.** If the user says "skip it for now," keep it as TODO with a note.

**Do not infer answers.** If the user's answer is ambiguous, ask a follow-up question.

**Keep the diff minimal.** Only change the sections directly related to the resolved TODOs. Do not reformat unrelated sections.

**If the document has no TODOs**, output: "対象ドキュメントにTODOは見つかりませんでした。" and stop.

---

## Forbidden — 絶対禁止事項

- **TODO を根拠なく削除禁止**: 「たぶん大丈夫」で TODO を消さない
- **推測での回答充填禁止**: 確認が取れていない内容を仕様として書かない
- **フォーマット変更禁止**: TODO 解決以外の場所を勝手に整形・リファクタしない
- **AI voice 禁止**: 「仕様書の品質向上のために〜」などの導入文を書かない
