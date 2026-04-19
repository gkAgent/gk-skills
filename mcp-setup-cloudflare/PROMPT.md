# MCP Cloudflare セットアップ

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/mcp-setup-cloudflare` で自動起動します（貼り付け不要）。

---

以下の指示に従って、Cloudflare MCP サーバーのセットアップを支援してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: 使用するCloudflareサービスの確認

まず以下を質問してください:

> Cloudflare の何を Claude Code から操作したいですか？（複数選択可）
>
> 1. **Workers** — デプロイ・ログ確認・コード読み取り
> 2. **D1** — SQLite データベースのクエリ・スキーマ確認
> 3. **R2** — オブジェクトストレージのファイル一覧・アップロード・削除
> 4. **KV** — Key-Value ストアの読み書き
> 5. **Pages** — ビルド・デプロイ状況確認
>
> また、**Account ID** は手元にありますか？
> （Cloudflare Dashboard → 右サイドバーに表示される 32文字の ID）

---

## Phase 1: API トークン取得と .mcp.json 生成

### Step 1 — API トークンを作成する

1. Cloudflare Dashboard → My Profile → **API Tokens** → Create Token
2. **Custom token** を選択
3. 以下の権限をチェック:

| 使いたいサービス | 必要な権限 |
|---|---|
| Workers 読み取り | `Workers Scripts: Read` |
| Workers デプロイ | `Workers Scripts: Edit` |
| D1 クエリ | `D1: Edit`（読み取りのみなら `Read`） |
| R2 読み取り | `Workers R2 Storage: Read` |
| R2 書き込み | `Workers R2 Storage: Edit` |
| KV 読み取り | `Workers KV Storage: Read` |
| KV 書き込み | `Workers KV Storage: Edit` |
| Pages 確認 | `Cloudflare Pages: Read` |

4. トークンを生成してコピーしておく

---

### Step 2 — .mcp.json を作成する

```json
{
  "mcpServers": {
    "cloudflare": {
      "command": "npx",
      "args": [
        "-y",
        "@cloudflare/mcp-server-cloudflare"
      ],
      "env": {
        "CLOUDFLARE_API_TOKEN": "your_api_token_here",
        "CLOUDFLARE_ACCOUNT_ID": "your_32char_account_id"
      }
    }
  }
}
```

**配置先:** プロジェクトルートの `.mcp.json`。

```bash
# .gitignore に追加
echo ".mcp.json" >> .gitignore
```

---

## Phase 2: 接続テスト

Claude Code を再起動して接続を確認してください。

```
# Claude Code のプロンプトで
Cloudflare アカウントに存在するWorkerの一覧を取得してください
```

**よくあるエラー:**

| エラー | 原因 | 対処 |
|---|---|---|
| `10000 Authentication error` | APIトークンが誤りまたは期限切れ | トークンを再発行 |
| `403 Forbidden` | 権限スコープ不足 | トークンに必要な権限を追加 |
| `Unknown account` | Account ID が間違い | Dashboard のサイドバーで再確認 |
| `npx: command not found` | Node.js 未インストール | `node -v` で確認 |

---

## Phase 3: 使用例

### Workers — デプロイ・確認

```
アカウントにある全Workerの一覧と、それぞれの最終デプロイ日時を教えてください
```

```
my-api というWorkerのコードを読んで、エンドポイント一覧をまとめてください
```

### D1 — データベース操作

```
D1データベース「prod-db」のテーブル一覧を取得して、各テーブルの行数を確認してください
```

### R2 — オブジェクトストレージ

```
R2バケット「uploads」の中にあるファイルをサイズの大きい順にTop10を表示してください
```

### KV — Key-Value ストア

```
KV namespace「APP_CONFIG」に存在するキーを全て一覧にしてください
```

---

## 絶対禁止事項

- **APIトークンを .mcp.json にベタ書きして公開リポジトリにコミットしない**: 必ず `.gitignore` に追加する
- **Edit権限を不必要に付けない**: 読み取りだけが目的なら `Read` スコープで十分
- **R2 の大量削除オペレーションを確認なしで実行させない**: Claude に削除前の確認ステップを明示的に指示すること
- **D1 の本番データへの書き込みは慎重に**: `Edit` 権限があると INSERT / UPDATE / DELETE も実行される
