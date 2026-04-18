---
name: mcp-setup-github
description: Sets up the GitHub MCP server for Claude Code. Guides through PAT creation with correct scopes, generates .mcp.json, and shows practical examples for Issues, Pull Requests, and GitHub Actions operations. Use when you want Claude Code to read/write GitHub resources, automate PR reviews, or query CI results without leaving the editor.
---

# MCP GitHub Setup

## Activation

When this skill is invoked, follow the exact flow below. Do not skip phases.

---

## Phase 0: Confirm Scope

Ask the user:

> GitHub MCP で何をしたいか教えてください。（複数選択可）
>
> 1. Issues の読み取り・作成・コメント
> 2. Pull Request の読み取り・作成・レビュー
> 3. リポジトリのファイル読み取り・コミット
> 4. GitHub Actions の実行状況確認
> 5. Releases の確認・作成
>
> また、対象リポジトリは **Public** ですか？ **Private** ですか？

選択内容に応じて、Phase 1 で適切な PAT スコープを案内する。

---

## Phase 1: PAT 取得と .mcp.json 生成

### Step 1 — PAT を作成する

1. GitHub → Settings → Developer settings → **Personal access tokens → Fine-grained tokens** を開く
2. **Generate new token** をクリック
3. 以下のスコープをチェック:

| やりたいこと | 必要なスコープ |
|---|---|
| Issues 読み書き | `issues: read/write` |
| PR 読み書き・レビュー | `pull_requests: read/write` |
| リポジトリ内容読み取り | `contents: read` |
| リポジトリ内容書き込み（コミット） | `contents: write` |
| Actions 確認 | `actions: read` |
| Releases 作成 | `contents: write` |

**最小権限の原則**: 使わないスコープはチェックしない。

4. Expiration は **90日** を推奨（無期限は避ける）
5. トークンを生成してコピーしておく（再表示できない）

---

### Step 2 — .mcp.json を配置する

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "github_pat_XXXXXXXXXXXX"
      }
    }
  }
}
```

**配置先:** プロジェクトルートの `.mcp.json` に追記、または新規作成。

**セキュリティ対応:**

```bash
# .gitignore に追加
echo ".mcp.json" >> .gitignore
```

---

## Phase 2: Connection Test

Claude Code を再起動して接続を確認する。

```
# Claude Code のプロンプトで
利用可能なGitHub MCPのツールを一覧にしてください
```

以下のようなツールが出力されれば成功:

- `github_list_issues`
- `github_get_pull_request`
- `github_create_issue`
- `github_list_commits`
- `github_get_file_contents`

**よくあるエラー:**

| エラー | 原因 | 対処 |
|---|---|---|
| `Bad credentials` | PAT が誤りまたは期限切れ | 新しいトークンを発行して再設定 |
| `Resource not accessible by integration` | スコープ不足 | PAT のスコープを見直す |
| `Not Found` | リポジトリ名またはオーナーが間違い | `owner/repo` の形式を確認 |
| `npx: command not found` | Node.js 未インストール | `node -v` で確認 |

---

## Phase 3: Usage Examples

### Issues 操作

```
このリポジトリの未クローズの Issues を一覧にして、
優先度順に並べてください
```

```
Issue #42 の内容を読んで、対応方針を提案してください
```

```
以下のバグ報告を Issue として作成してください:
タイトル: ログイン画面でパスワードが空でも送信できる
本文: 再現手順 / 期待動作 / 実際の動作
```

### Pull Request 操作

```
open状態のPRを一覧にして、各PRのレビュー状況を教えてください
```

```
PR #15 の差分を見て、コードレビューのコメントを書いてください
特にセキュリティとパフォーマンスの観点で確認してください
```

```
feature/add-login ブランチからmainへのPRを作成してください
タイトルと説明は差分から自動生成してください
```

### GitHub Actions 確認

```
最新のワークフロー実行結果を確認して、
失敗しているジョブがあれば原因を教えてください
```

### ファイル読み取り

```
src/auth/login.ts を GitHub から読み込んで、
脆弱性がないか確認してください
```

---

## Forbidden — 禁止事項

- **PAT を .mcp.json に直書きして公開リポジトリにコミットしない**: 必ず `.gitignore` に追加するか、環境変数（`$GITHUB_TOKEN`）を使う
- **`contents: write` スコープを不必要に付けない**: 読み取りのみなら `contents: read` で十分
- **Organization の PAT は IT/セキュリティ部門の承認を得てから使う**: Fine-grained token の Organization アクセスは管理者承認が必要な場合がある
