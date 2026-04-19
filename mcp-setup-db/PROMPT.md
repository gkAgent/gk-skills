# MCP データベース セットアップ

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/mcp-setup-db` で自動起動します（貼り付け不要）。

---

以下の指示に従って、データベース MCP サーバーのセットアップを支援してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: データベースの確認

まず以下を質問してください:

> 接続したいデータベースを教えてください。
>
> **DB**: PostgreSQL / MySQL / SQLite
>
> **接続情報**（後で伏せ字にします）:
> - PostgreSQL / MySQL: host, port, dbname, user, password
> - SQLite: `.db` ファイルの絶対パス

すでにコンテキストが提供されている場合は Phase 1 に進んでください。

---

## Phase 1: .mcp.json の生成

### PostgreSQL

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://USER:${PG_PASSWORD}@HOST:PORT/DBNAME"
      ],
      "env": {
        "PG_PASSWORD": "your_password_here"
      }
    }
  }
}
```

---

### MySQL

```json
{
  "mcpServers": {
    "mysql": {
      "command": "npx",
      "args": ["-y", "mcp-server-mysql"],
      "env": {
        "MYSQL_HOST": "HOST",
        "MYSQL_PORT": "3306",
        "MYSQL_DATABASE": "DBNAME",
        "MYSQL_USER": "USER",
        "MYSQL_PASSWORD": "PASSWORD"
      }
    }
  }
}
```

---

### SQLite

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sqlite",
        "--db-path",
        "/absolute/path/to/your.db"
      ]
    }
  }
}
```

**配置先:** プロジェクトルートに `.mcp.json` として保存してください。

---

## Phase 2: 接続テスト

設定ファイル保存後、Claude Code を再起動して接続を確認してください。

```
# Claude Code のプロンプトで
利用可能なMCPツールを一覧表示してください
```

**よくある接続エラーと対処:**

| エラー | 原因 | 対処 |
|---|---|---|
| `ECONNREFUSED` | ホスト/ポート間違い、またはDB未起動 | 接続情報を再確認 |
| `password authentication failed` | パスワード誤り | パスワードを再確認。特殊文字はURLエンコード |
| `database does not exist` | DB名間違い | `psql -l` / `SHOW DATABASES;` で確認 |
| `permission denied` | 権限不足 | DBユーザーにSELECT権限を付与 |
| `npx: command not found` | Node.js未インストール | `node -v` で確認 |

---

## Phase 3: 使用例

### スキーマ確認

```
usersテーブルの定義を見せてください
```

```
このDBに存在するテーブルを一覧にして、それぞれの用途を推測してください
```

### クエリ実行・デバッグ

```
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id
このクエリの実行結果と、N+1になっていないか確認してください
```

### データ調査

```
ordersテーブルで先月のデータが異常に少ない。何が起きているか調べてください
```

### マイグレーション支援

```
現在のスキーマを見て、Prismaのschema.prismaを生成してください
```

---

## 絶対禁止事項

- **本番DBに直接書き込むSQLをClaudeに実行させない**: MCPサーバーはデフォルトでDML実行可能。本番環境では読み取り専用ユーザーを使う
- **パスワードをargs配列にベタ書きしない**: `.mcp.json` をgitにコミットするなら必ず `env` 変数化し、`.gitignore` に追加する
- **.mcp.json を公開リポジトリにコミットしない**: secrets が含まれる場合は `.gitignore` に追加すること
