# MCP データベース セットアップ — 全DB対応版

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/mcp-setup-db-all` で自動起動します（貼り付け不要）。

---

以下の指示に従って、データベース MCP サーバーのセットアップを支援してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: データベースの選択

まず以下を質問してください:

> 接続したいデータベースを選んでください（複数可）。
>
> **リレーショナル DB:**
> - PostgreSQL / MySQL / MariaDB / SQLite / SQL Server (MSSQL) / Oracle
>
> **NoSQL / キャッシュ:**
> - MongoDB / Redis
>
> 複数を選んだ場合は、それぞれの接続情報を教えてください。
> パスワードなどの機密情報は、生成後すぐに `.gitignore` に追加します。

すでにコンテキストが提供されている場合は Phase 1 に進んでください。

---

## Phase 1: .mcp.json の生成

選択された DB に対応するブロックを組み合わせて `.mcp.json` を生成してください。

### PostgreSQL

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://USER:PASSWORD@HOST:5432/DBNAME"]
    }
  }
}
```

### MySQL / MariaDB

```json
{
  "mcpServers": {
    "mysql": {
      "command": "npx",
      "args": ["-y", "mysql-mcp-server"],
      "env": {
        "MYSQL_HOST": "localhost", "MYSQL_PORT": "3306",
        "MYSQL_USER": "USER", "MYSQL_PASSWORD": "PASSWORD", "MYSQL_DATABASE": "DBNAME"
      }
    }
  }
}
```

**注意:** パスワードを `args` に直書きせず `env` を使う（`ps aux` で見える問題を回避）。

### SQLite

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "/absolute/path/to/database.db"]
    }
  }
}
```

**注意:** パスは絶対パスが必須。相対パスは起動ディレクトリ依存でトラブルの元。

### SQL Server (MSSQL)

```json
{
  "mcpServers": {
    "mssql": {
      "command": "npx",
      "args": ["-y", "mssql-mcp-server"],
      "env": {
        "MSSQL_HOST": "localhost", "MSSQL_PORT": "1433",
        "MSSQL_USER": "sa", "MSSQL_PASSWORD": "PASSWORD", "MSSQL_DATABASE": "DBNAME",
        "MSSQL_ENCRYPT": "false", "MSSQL_TRUST_SERVER_CERTIFICATE": "true"
      }
    }
  }
}
```

**注意:** Azure SQL の場合は `MSSQL_ENCRYPT=true`、`MSSQL_TRUST_SERVER_CERTIFICATE=false`。

### Oracle

```json
{
  "mcpServers": {
    "oracle": {
      "command": "npx",
      "args": ["-y", "oracle-mcp-server"],
      "env": {
        "ORACLE_USER": "USER", "ORACLE_PASSWORD": "PASSWORD",
        "ORACLE_CONNECTION_STRING": "localhost:1521/ORCL"
      }
    }
  }
}
```

**注意:** Oracle Instant Client が事前インストール必須。接続文字列は `HOST:PORT/SERVICE_NAME` 形式。

### MongoDB

```json
{
  "mcpServers": {
    "mongodb": {
      "command": "npx",
      "args": ["-y", "mongodb-mcp-server", "--connectionString", "mongodb://USER:PASSWORD@localhost:27017/DBNAME?authSource=admin"]
    }
  }
}
```

### Redis

```json
{
  "mcpServers": {
    "redis": {
      "command": "npx",
      "args": ["-y", "redis-mcp-server"],
      "env": {
        "REDIS_HOST": "localhost", "REDIS_PORT": "6379",
        "REDIS_PASSWORD": "PASSWORD", "REDIS_DB": "0"
      }
    }
  }
}
```

---

## Phase 2: 接続テスト

生成した `.mcp.json` を保存後、Claude Code を再起動して以下のプロンプトを実行してください:

```
# PostgreSQL / MySQL / SQLite / MSSQL / Oracle
テーブル一覧を表示してください。

# MongoDB
コレクション一覧を表示してください。

# Redis
全キーを一覧表示してください（KEYS *）。
```

### よくあるエラーと対処

| エラー | 原因 | 対処 |
|---|---|---|
| `ECONNREFUSED` | DB が起動していない / ポート違い | DB プロセス確認・ポート確認 |
| `authentication failed` | ユーザー名/パスワード誤り | DB側のユーザー権限確認 |
| Oracle `ORA-12154` | TNS 解決失敗 | Instant Client の `tnsnames.ora` 確認 |
| MongoDB `AuthenticationFailed` | authSource 不一致 | `?authSource=admin` の有無確認 |
| Redis `NOAUTH` | パスワード設定済みなのに未指定 | `REDIS_PASSWORD` を設定 |
| `npx: command not found` | Node.js 未インストール | Node.js 18+ をインストール |

---

## Phase 3: 実際の使い方例

```
usersテーブルのカラム定義とインデックスをすべて教えてください。
```

```
以下のクエリの実行計画を取得して、改善案を提示してください。
SELECT * FROM orders WHERE customer_id = 123 ORDER BY created_at DESC;
```

```
PostgreSQLのsessionsテーブルとRedisのsession:*キーを照合して、
DBには存在するがRedisに存在しないセッションIDを抽出してください。
```

---

## .gitignore への追加（必須）

```
# .gitignore
.mcp.json
.claude.json
.env
.env.local
```

---

## 絶対禁止事項

- `.mcp.json` に接続文字列を貼った後、長々とした警告を書かない。`.gitignore` に追加するよう1行で指示する
- DB が起動していなかった場合、原因推測を並べない。確認コマンドを1つ示す
