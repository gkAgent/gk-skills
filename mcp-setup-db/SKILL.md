---
name: mcp-setup-db
description: Sets up a database MCP server (PostgreSQL, MySQL, or SQLite) for Claude Code. Generates a valid .mcp.json configuration, runs a connection test, and shows practical query examples. Use when you want Claude Code to query your database directly, inspect schemas, or debug SQL without leaving the editor.
---

# MCP Database Setup

## Activation

When this skill is invoked, follow the exact flow below. Do not skip phases.

---

## Phase 0: Choose Your Database

Ask the user:

> 接続したいデータベースを教えてください。
>
> **DB**: PostgreSQL / MySQL / SQLite
>
> **接続情報**（後で伏せ字にします）:
> - PostgreSQL / MySQL: host, port, dbname, user, password
> - SQLite: `.db` ファイルの絶対パス

If the user has already provided this context, skip to Phase 1.

---

## Phase 1: Generate .mcp.json

### PostgreSQL

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://USER:PASSWORD@HOST:PORT/DBNAME"
      ]
    }
  }
}
```

**置き換え手順:**
1. `USER` → DBユーザー名
2. `PASSWORD` → パスワード（後述のenv変数化を推奨）
3. `HOST` → ホスト名またはIPアドレス
4. `PORT` → ポート番号（デフォルト: `5432`）
5. `DBNAME` → データベース名

**パスワードをenv変数化する場合（推奨）:**

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
      "args": [
        "-y",
        "mcp-server-mysql"
      ],
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

**配置先:** プロジェクトルートに `.mcp.json` として保存する。

---

## Phase 2: Connection Test

設定ファイル保存後、Claude Code を再起動して接続を確認する。

**確認手順:**

```
# Claude Code のプロンプトで
利用可能なMCPツールを一覧表示してください
```

以下のようなツールが表示されれば成功:
- PostgreSQL: `postgres_query`, `postgres_list_tables`
- MySQL: `mysql_query`, `mysql_list_tables`
- SQLite: `sqlite_query`, `sqlite_list_tables`

**よくある接続エラーと対処:**

| エラー | 原因 | 対処 |
|---|---|---|
| `ECONNREFUSED` | ホスト/ポート間違い、またはDB未起動 | 接続情報を再確認。`telnet HOST PORT` で疎通確認 |
| `password authentication failed` | パスワード誤り | パスワードを再確認。特殊文字はURLエンコード (`@`→`%40`) |
| `database does not exist` | DB名間違い | `psql -l` / `SHOW DATABASES;` で実在するDB名を確認 |
| `permission denied` | 権限不足 | DBユーザーにSELECT権限を付与 |
| `npx: command not found` | Node.js未インストール | `node -v` で確認。未インストールなら [nodejs.org](https://nodejs.org) から導入 |

---

## Phase 3: Usage Examples

接続が確認できたら、以下のプロンプトで使い始める。

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

## Forbidden — 禁止事項

- **本番DBに直接書き込むSQLをClaudeに実行させない**: MCPサーバーはデフォルトでDML実行可能。本番環境では読み取り専用ユーザーを使う
- **パスワードをargs配列にベタ書きしない**: `.mcp.json` をgitにコミットするなら必ず `env` 変数化し、`.gitignore` に追加する
- **.mcp.json を公開リポジトリにコミットしない**: secrets が含まれる場合は `.gitignore` に追加すること
