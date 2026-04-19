# MCP コミュニケーションツール セットアップ (Slack / LINE / Discord)

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/mcp-setup-communication` で自動起動します（貼り付け不要）。

---

以下の指示に従って、コミュニケーションツールの MCP サーバーをセットアップしてください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: プラットフォームの選択

まず以下を質問してください:

> 連携したいコミュニケーションツールを教えてください。
>
> **ツール**: Slack / LINE / Discord
>
> また、主にどんな用途で使いたいですか？（複数可）
>
> 1. アプリのエラー通知を自動送信したい
> 2. CI/CDの結果を通知したい
> 3. チャンネル / グループのメッセージを読み取りたい
> 4. 特定のメッセージにリアクション / 返信したい

---

## Phase 1: セットアップと .mcp.json 生成

### Slack

**Step 1 — Bot Token を取得する**

1. [api.slack.com/apps](https://api.slack.com/apps) → **Create New App** → **From scratch**
2. **OAuth & Permissions** → Bot Token Scopes に以下を追加:

| やりたいこと | 必要なスコープ |
|---|---|
| メッセージ送信 | `chat:write` |
| チャンネル一覧取得 | `channels:read` |
| メッセージ読み取り | `channels:history` |
| DM送信 | `im:write` |

3. **Install to Workspace** → Bot User OAuth Token をコピー（`xoxb-` で始まる）

**Step 2 — .mcp.json を作成する**

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-XXXXXXXXXXXX-XXXXXXXXXXXX-XXXXXXXXXXXXXXXXXXXXXXXX",
        "SLACK_TEAM_ID": "TXXXXXXXXXX"
      }
    }
  }
}
```

---

### LINE

**Step 1 — Messaging API のトークンを取得する**

1. [LINE Developers Console](https://developers.line.biz/console/) → Provider → Create a new channel → **Messaging API**
2. **Messaging API** タブ → **Channel access token** → Issue

**Step 2 — .mcp.json を作成する**

```json
{
  "mcpServers": {
    "line": {
      "command": "npx",
      "args": ["-y", "mcp-server-line"],
      "env": {
        "LINE_CHANNEL_ACCESS_TOKEN": "your_channel_access_token_here",
        "LINE_CHANNEL_SECRET": "your_channel_secret_here"
      }
    }
  }
}
```

---

### Discord

**Step 1 — Bot Token を取得する**

1. [discord.com/developers/applications](https://discord.com/developers/applications) → **New Application**
2. **Bot** タブ → Token をコピー
3. **OAuth2 → URL Generator** でスコープ選択 → 生成 URL でサーバーに Bot を招待

**Step 2 — .mcp.json を作成する**

```json
{
  "mcpServers": {
    "discord": {
      "command": "npx",
      "args": ["-y", "mcp-server-discord"],
      "env": {
        "DISCORD_TOKEN": "your_bot_token_here"
      }
    }
  }
}
```

**全プラットフォーム共通:**

```bash
# .gitignore に追加
echo ".mcp.json" >> .gitignore
```

---

## Phase 2: 接続テスト

Claude Code を再起動して接続を確認してください。

**Slack:** `Slackのチャンネル一覧を取得してください`
**LINE:** `LINEのBotプロファイル情報を取得してください`
**Discord:** `Discordのサーバー（Guild）一覧を取得してください`

**よくあるエラー:**

| エラー | 原因 | 対処 |
|---|---|---|
| `invalid_auth` (Slack) | Bot Token が誤りまたは期限切れ | Token を再発行 |
| `missing_scope` (Slack) | スコープ不足 | App の OAuth Scopes を追加して再インストール |
| `401 Unauthorized` (LINE) | Channel Access Token が誤り | Console で再発行 |
| `TOKEN_INVALID` (Discord) | Bot Token が誤り | Developer Portal で再生成 |

---

## Phase 3: 使用例

### Slack — 通知・メッセージ操作

```
#alerts チャンネルに「本番デプロイ完了: v2.3.1」を送信してください
```

```
#general チャンネルの直近20件のメッセージを読んで、未解決の質問があれば要約してください
```

### LINE — 通知送信

```
LINE の UserID: Uxxxxx に「[Alert] サーバーのCPU使用率が90%を超えました」を送信してください
```

### Discord — サーバー・チャンネル操作

```
#bot-test チャンネルに「テスト通知: MCP接続確認OK」を送信してください
```

---

## 絶対禁止事項

- **Bot Token / Access Token を .mcp.json にベタ書きして公開リポジトリにコミットしない**: 必ず `.gitignore` に追加する
- **メンション（@channel / @here / @everyone）を自動で送信させない**: 意図しない大量通知が発生する
- **ユーザーのDMを無断で読み取る操作をさせない**: プライバシーポリシーおよび各プラットフォームの利用規約に反する
- **LINE の場合、個人ユーザーへの大量メッセージ送信はレート制限に注意**: 月間無料枠（200通）を超えると課金される
