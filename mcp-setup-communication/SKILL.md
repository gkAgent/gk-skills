---
name: mcp-setup-communication
description: Sets up a communication platform MCP server (Slack, LINE, or Discord) for Claude Code. Guides through bot token setup, .mcp.json generation, and message send/receive examples with notification configuration. Use when you want Claude Code to post alerts, read channel messages, or automate team notifications directly from your development workflow.
---

# MCP Communication Setup (Slack / LINE / Discord)

## Activation

When this skill is invoked, follow the exact flow below. Do not skip phases.

---

## Phase 0: Choose Your Platform

Ask the user:

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
2. App Name と Workspace を設定
3. **OAuth & Permissions** → Scopes → **Bot Token Scopes** に以下を追加:

| やりたいこと | 必要なスコープ |
|---|---|
| メッセージ送信 | `chat:write` |
| チャンネル一覧取得 | `channels:read` |
| メッセージ読み取り | `channels:history` |
| DM送信 | `im:write` |
| ファイル投稿 | `files:write` |
| リアクション追加 | `reactions:write` |

4. **Install to Workspace** → Bot User OAuth Token をコピー（`xoxb-` で始まる）

**Step 2 — .mcp.json を作成する**

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-slack"
      ],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-XXXXXXXXXXXX-XXXXXXXXXXXX-XXXXXXXXXXXXXXXXXXXXXXXX",
        "SLACK_TEAM_ID": "TXXXXXXXXXX"
      }
    }
  }
}
```

**Team ID の確認方法:** Slack を開き、ワークスペース名をクリック → Settings → 下部の「Workspace ID」。

---

### LINE

**Step 1 — Messaging API のチャネルとトークンを取得する**

1. [LINE Developers Console](https://developers.line.biz/console/) → Provider → **Create a new channel** → **Messaging API**
2. チャネル設定 → **Messaging API** タブ → **Channel access token** → Issue

**Step 2 — .mcp.json を作成する**

```json
{
  "mcpServers": {
    "line": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-server-line"
      ],
      "env": {
        "LINE_CHANNEL_ACCESS_TOKEN": "your_channel_access_token_here",
        "LINE_CHANNEL_SECRET": "your_channel_secret_here"
      }
    }
  }
}
```

**送信先の User ID または Group ID を取得する方法:**

```
# Claude Code のプロンプトで
LINEボットに送ったメッセージのsenderのuserIdを取得してください
```

ボットにメッセージを送ると Webhook イベントに `userId` / `groupId` が含まれる。

---

### Discord

**Step 1 — Bot Token を取得する**

1. [discord.com/developers/applications](https://discord.com/developers/applications) → **New Application**
2. **Bot** タブ → **Add Bot** → Token をコピー
3. **OAuth2 → URL Generator** で以下のスコープを選択:
   - `bot`
   - 必要な権限: `Read Messages/View Channels`, `Send Messages`, `Read Message History`
4. 生成された URL でサーバーに Bot を招待

**Step 2 — .mcp.json を作成する**

```json
{
  "mcpServers": {
    "discord": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-server-discord"
      ],
      "env": {
        "DISCORD_TOKEN": "your_bot_token_here"
      }
    }
  }
}
```

**配置先（全プラットフォーム共通）:** プロジェクトルートの `.mcp.json`。

```bash
# .gitignore に追加
echo ".mcp.json" >> .gitignore
```

---

## Phase 2: Connection Test

Claude Code を再起動して接続を確認する。

**Slack:**

```
# Claude Code のプロンプトで
Slackのチャンネル一覧を取得してください
```

**LINE:**

```
LINEのBotプロファイル情報を取得してください
```

**Discord:**

```
Discordのサーバー（Guild）一覧を取得してください
```

**よくあるエラー:**

| エラー | 原因 | 対処 |
|---|---|---|
| `invalid_auth` (Slack) | Bot Token が誤りまたは期限切れ | Token を再発行 |
| `missing_scope` (Slack) | スコープ不足 | App の OAuth Scopes を追加して再インストール |
| `401 Unauthorized` (LINE) | Channel Access Token が誤り | Console で再発行 |
| `TOKEN_INVALID` (Discord) | Bot Token が誤り | Developer Portal で再生成 |
| Bot がチャンネルにいない | Bot を招待していない | チャンネルに Bot を追加する |

---

## Phase 3: Usage Examples

### Slack — 通知・メッセージ操作

```
#alerts チャンネルに以下のメッセージを送信してください:
「本番デプロイ完了: v2.3.1 / 2024-01-15 14:32 JST」
```

```
#general チャンネルの直近20件のメッセージを読んで、
未解決の質問があれば要約してください
```

```
CI/CD が失敗したら #dev-alerts に通知するメッセージの
テンプレートを作って送信テストしてください
```

### LINE — 通知送信

```
LINE の UserID: Uxxxxx に
「[Alert] サーバーのCPU使用率が90%を超えました。確認してください。」
を送信してください
```

### Discord — サーバー・チャンネル操作

```
#bot-test チャンネルに「テスト通知: MCP接続確認OK」を送信してください
```

```
#announcements の直近10件の投稿を要約してください
```

---

## Forbidden — 禁止事項

- **Bot Token / Access Token を .mcp.json にベタ書きして公開リポジトリにコミットしない**: 必ず `.gitignore` に追加する
- **メンション（@channel / @here / @everyone）を自動で送信させない**: 意図しない大量通知が発生する
- **ユーザーのDMを無断で読み取る操作をさせない**: プライバシーポリシーおよび各プラットフォームの利用規約に反する可能性がある
- **LINE の場合、個人ユーザーへの大量メッセージ送信はレート制限に注意**: Messaging API の月間無料枠（200通）を超えると課金される
