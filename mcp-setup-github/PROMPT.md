# MCP GitHub セットアップ

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/mcp-setup-github` で自動起動します（貼り付け不要）。

---

以下の指示に従って、GitHub MCP サーバーのセットアップを支援してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> GitHub MCP で何をしたいか教えてください。（複数選択可）
>
> 1. Issues の読み取り・作成・コメント
> 2. Pull Request の読み取り・作成・レビュー
> 3. リポジトリのファイル読み取り・コミット
> 4. GitHub Actions の実行状況確認
> 5. Releases の確認・作成
>
> また、対象リポジトリは **Public** ですか？ **Private** ですか？

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
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "github_pat_XXXXXXXXXXXX"
      }
    }
  }
}
```

**配置先:** プロジェクトルートの `.mcp.json` に追記、または新規作成。

```bash
# .gitignore に追加
echo ".mcp.json" >> .gitignore
```

---

## Phase 2: 接続テスト

Claude Code を再起動して接続を確認してください。

```
# Claude Code のプロンプトで
利用可能なGitHub MCPのツールを一覧にしてください
```

**よくあるエラー:**

| エラー | 原因 | 対処 |
|---|---|---|
| `Bad credentials` | PAT が誤りまたは期限切れ | 新しいトークンを発行して再設定 |
| `Resource not accessible by integration` | スコープ不足 | PAT のスコープを見直す |
| `Not Found` | リポジトリ名またはオーナーが間違い | `owner/repo` の形式を確認 |
| `npx: command not found` | Node.js 未インストール | `node -v` で確認 |

---

## Phase 3: 使用例

### Issues 操作

```
このリポジトリの未クローズの Issues を一覧にして、優先度順に並べてください
```

```
以下のバグ報告を Issue として作成してください:
タイトル: ログイン画面でパスワードが空でも送信できる
本文: 再現手順 / 期待動作 / 実際の動作
```

### Pull Request 操作

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
最新のワークフロー実行結果を確認して、失敗しているジョブがあれば原因を教えてください
```

---

## 絶対禁止事項

- **PAT を .mcp.json に直書きして公開リポジトリにコミットしない**: 必ず `.gitignore` に追加するか、環境変数を使う
- **`contents: write` スコープを不必要に付けない**: 読み取りのみなら `contents: read` で十分
- **Organization の PAT は IT/セキュリティ部門の承認を得てから使う**: Fine-grained token の Organization アクセスは管理者承認が必要な場合がある
