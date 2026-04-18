---
name: mcp-audit
description: Audits existing .mcp.json and .claude.json MCP configurations for security risks, permission issues, and quality problems. Detects exposed tokens, overly broad scopes, redundant entries, and outdated server versions. Outputs a structured report with severity ratings and specific remediation steps. Use when you want to review MCP configs before committing, after onboarding a new team member, or as part of a security audit.
---

# MCP Configuration Audit

## Activation

When this skill is invoked, follow the exact flow below. Do not skip phases.

---

## Phase 0: Locate Configuration Files

Ask the user:

> 診断対象のファイルを教えてください。以下から該当するものを選んでください。
>
> 1. プロジェクトルートの `.mcp.json`
> 2. ホームディレクトリの `.claude.json`（`~/.claude.json`）
> 3. 両方
> 4. 別のパス（パスを教えてください）
>
> ファイルの内容を貼り付けるか、`@.mcp.json` で参照してください。

ファイルが提供されたら Phase 1 に進む。

---

## Phase 1: Security Risk Scan

以下のチェックリストを **全項目** 確認する。スキップしない。

### S1 — トークン・シークレット露出チェック

各 `mcpServers` エントリの `args` 配列と `env` フィールドを走査する。

**検出パターン:**

| パターン | リスクレベル | 説明 |
|---|---|---|
| `args` 配列にトークン文字列が直書き | 🔴 HIGH | git 履歴に残る。即時対応が必要 |
| `env` フィールドにトークンが直書き | 🟡 MEDIUM | `.mcp.json` が `.gitignore` 対象なら許容範囲。要確認 |
| 接続文字列にパスワードが含まれる | 🔴 HIGH | `postgresql://user:PASSWORD@host/db` 形式 |
| `${}` / `${VAR}` で環境変数参照 | ✅ OK | 推奨パターン |

**確認事項:**

```
# .mcp.json が .gitignore に含まれているか
.gitignore に .mcp.json の記載があるか確認してください
```

---

### S2 — 過剰権限チェック

各 MCP サーバーの設定から、以下の過剰権限パターンを検出する。

| サービス | 過剰なパターン | 推奨 |
|---|---|---|
| GitHub | `contents: write` が不要な場合 | 読み取りのみなら `contents: read` |
| GitHub | repo 全体アクセス vs. 特定リポジトリ | Fine-grained token で絞り込む |
| Cloudflare | `Edit` 権限が全サービスに付いている | 使わないサービスは `Read` に |
| Datadog | Application Key に全権限 | 使用するスコープのみ付与 |
| Slack | `admin` スコープが含まれる | 不要なら削除 |
| DB | 書き込み可能なユーザーで本番DBに接続 | 読み取り専用ユーザーを別途作成 |

---

### S3 — 設定品質チェック

| チェック項目 | 問題のあるパターン |
|---|---|
| 重複エントリ | 同じサービスが複数の `mcpServers` キーで定義されている |
| 未使用エントリ | 削除されたプロジェクトのMCP設定が残っている |
| `command` のパス問題 | 絶対パスが別環境で動かない（`/Users/xxx/...` など） |
| `npx` でバージョン未固定 | `-y` のみで最新版を使うと破壊的変更を拾う可能性 |
| タイムアウト設定なし | 長時間クエリで Claude Code がハングする可能性 |

---

### S4 — バージョン陳腐化チェック

`args` の npm パッケージ名を確認し、固定バージョンなしで `-y` のみ指定されているものを検出する。

**検出パターン:**

```json
"args": ["-y", "@modelcontextprotocol/server-postgres"]
```
→ バージョン未固定。npm の最新版が自動適用される。

**推奨パターン:**

```json
"args": ["-y", "@modelcontextprotocol/server-postgres@0.6.2"]
```

---

## Phase 2: Report Generation

Phase 1 の結果を以下のフォーマットで出力する。

```markdown
# MCP 設定診断レポート

診断日時: YYYY-MM-DD HH:MM
対象ファイル: .mcp.json / .claude.json

---

## サマリー

| 重要度 | 件数 |
|---|---|
| 🔴 HIGH（即時対応） | X 件 |
| 🟡 MEDIUM（近日対応） | X 件 |
| 🟢 LOW（改善推奨） | X 件 |
| ✅ 問題なし | X 件 |

---

## 検出事項

### [HIGH-001] トークン直書き — postgres サーバー

**場所:** `mcpServers.postgres.args[2]`
**内容:** 接続文字列にパスワードが直書きされています
**リスク:** .mcp.json が git に含まれると認証情報が漏洩します

**修正方法:**
```json
// 変更前
"args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://user:PASSWORD@host/db"]

// 変更後
"args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://user:${PG_PASSWORD}@host/db"],
"env": {
  "PG_PASSWORD": "PASSWORD"
}
```

また、`.gitignore` に `.mcp.json` を追加してください。

---

### [MEDIUM-001] 過剰スコープ — github サーバー

**場所:** GitHub PAT の設定（GITHUB_PERSONAL_ACCESS_TOKEN）
**内容:** `contents: write` が設定されていますが、読み取りのみの用途で使用されています
**リスク:** トークン漏洩時の影響範囲が拡大します

**修正方法:** GitHub Settings → API Tokens から `contents: read` のみのトークンを再発行してください。

---

### [LOW-001] バージョン未固定 — slack サーバー

**場所:** `mcpServers.slack.args`
**内容:** `@modelcontextprotocol/server-slack` のバージョンが固定されていません
**リスク:** npm の自動アップデートで破壊的変更を拾う可能性があります

**修正方法:**
```json
"args": ["-y", "@modelcontextprotocol/server-slack@1.0.0"]
```
現在の最新バージョンを `npm info @modelcontextprotocol/server-slack version` で確認してください。

---

## 問題なし

- ✅ S2-github: `.mcp.json` は `.gitignore` に含まれています
- ✅ S3-redundancy: 重複エントリは検出されませんでした
```

---

## Phase 3: Remediation Guidance

レポート出力後、ユーザーに優先順位を確認する。

> 上記のレポートを確認してください。
>
> 修正を進める場合:
> - 🔴 HIGH の項目から順に対応することを推奨します
> - 修正後に `.mcp.json` を再共有いただければ、再診断します
>
> どの項目から修正を始めますか？

ユーザーが特定の項目を選んだ場合、修正後の設定を完全なコードブロックで提示する。

---

## Forbidden — 禁止事項

- **診断せずに問題なしと報告しない**: 全 S1〜S4 チェックを完了してからレポートを出力する
- **検出した secrets を出力に含めない**: トークン値・パスワードはマスク（`****`）して表示する
- **修正方法を提示せず問題だけ報告しない**: 各検出事項には必ず具体的な修正手順を付ける
- **`.gitignore` 確認を省略しない**: S1 の判定は `.gitignore` の状態に依存するため、必ず確認する
