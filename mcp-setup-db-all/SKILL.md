---
name: mcp-setup-db-all
description: Sets up MCP server connections for ALL major databases in one skill — PostgreSQL, MySQL, SQLite, SQL Server, Oracle, MongoDB, Redis. Generates .mcp.json for single or multi-DB configurations, runs connection tests, and shows query examples per database. Use when your project uses multiple databases, when you want Claude Code to query any database directly, or when you want to compare schemas across different DB systems.
---

# MCP Database Setup — All Databases

## Activation

When this skill is invoked, follow the exact flow below. Do not skip phases.

---

## Phase 0: Choose Your Database(s)

Ask the user:

> 接続したいデータベースを選んでください（複数可）。
>
> **リレーショナル DB:**
> - PostgreSQL
> - MySQL / MariaDB
> - SQLite
> - SQL Server (MSSQL)
> - Oracle
>
> **NoSQL / キャッシュ:**
> - MongoDB
> - Redis
>
> 複数を選んだ場合は、それぞれの接続情報を教えてください。
> パスワードなどの機密情報は、生成後すぐに `.gitignore` に追加します。

If the user has already provided this context, skip to Phase 1.

---

## Phase 1: Generate .mcp.json

選択された DB に対応するブロックを組み合わせて `.mcp.json` を生成する。

### PostgreSQL

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://USER:PASSWORD@HOST:5432/DBNAME"
      ]
    }
  }
}
```

**インストール不要**（npx が自動取得）。Archived だが動作に問題なし。

---

### MySQL / MariaDB

```json
{
  "mcpServers": {
    "mysql": {
      "command": "npx",
      "args": ["-y", "mysql-mcp-server"],
      "env": {
        "MYSQL_HOST": "localhost",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "USER",
        "MYSQL_PASSWORD": "PASSWORD",
        "MYSQL_DATABASE": "DBNAME"
      }
    }
  }
}
```

**注意:** パスワードを `args` に直書きせず `env` を使う（`ps aux` で見える問題を回避）。

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
        "/absolute/path/to/database.db"
      ]
    }
  }
}
```

**注意:** パスは絶対パスが必須。相対パスは起動ディレクトリ依存でトラブルの元。

---

### SQL Server (MSSQL)

```json
{
  "mcpServers": {
    "mssql": {
      "command": "npx",
      "args": ["-y", "mssql-mcp-server"],
      "env": {
        "MSSQL_HOST": "localhost",
        "MSSQL_PORT": "1433",
        "MSSQL_USER": "sa",
        "MSSQL_PASSWORD": "PASSWORD",
        "MSSQL_DATABASE": "DBNAME",
        "MSSQL_ENCRYPT": "false",
        "MSSQL_TRUST_SERVER_CERTIFICATE": "true"
      }
    }
  }
}
```

**注意:** Azure SQL の場合は `MSSQL_ENCRYPT=true`、`MSSQL_TRUST_SERVER_CERTIFICATE=false`。
Windows 認証を使う場合は `MSSQL_TRUSTED_CONNECTION=true` を追加。

---

### Oracle

```json
{
  "mcpServers": {
    "oracle": {
      "command": "npx",
      "args": ["-y", "oracle-mcp-server"],
      "env": {
        "ORACLE_USER": "USER",
        "ORACLE_PASSWORD": "PASSWORD",
        "ORACLE_CONNECTION_STRING": "localhost:1521/ORCL"
      }
    }
  }
}
```

**注意:** Oracle Instant Client が事前インストール必須。
`LD_LIBRARY_PATH`（Linux）または `PATH`（Windows）に Instant Client のパスを追加してから起動すること。
接続文字列は `HOST:PORT/SERVICE_NAME` または `HOST:PORT:SID` の2形式。

---

### MongoDB

```json
{
  "mcpServers": {
    "mongodb": {
      "command": "npx",
      "args": [
        "-y",
        "mongodb-mcp-server",
        "--connectionString",
        "mongodb://USER:PASSWORD@localhost:27017/DBNAME?authSource=admin"
      ]
    }
  }
}
```

**注意:** MongoDB Atlas の場合は接続文字列を Atlas の "Connect" から取得。
`mongodb+srv://` 形式もそのまま使える。

---

### Redis

```json
{
  "mcpServers": {
    "redis": {
      "command": "npx",
      "args": ["-y", "redis-mcp-server"],
      "env": {
        "REDIS_HOST": "localhost",
        "REDIS_PORT": "6379",
        "REDIS_PASSWORD": "PASSWORD",
        "REDIS_DB": "0"
      }
    }
  }
}
```

**注意:** パスワードなし（開発環境）の場合は `REDIS_PASSWORD` を省略。
Redis 6.0+ の ACL を使う場合は `REDIS_USERNAME` を追加。

---

### 複数 DB の組み合わせ例（Web アプリ典型構成）

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://app_user:PASSWORD@localhost:5432/appdb"
      ]
    },
    "redis": {
      "command": "npx",
      "args": ["-y", "redis-mcp-server"],
      "env": {
        "REDIS_HOST": "localhost",
        "REDIS_PORT": "6379",
        "REDIS_DB": "0"
      }
    }
  }
}
```

複数 DB を同時に接続すると、Claude が SQL クエリ・キャッシュ・スキーマを横断的に参照できる。

---

## Phase 2: 接続テスト

生成した `.mcp.json` を `.claude.json` または `.mcp.json` として保存後、Claude Code を再起動して以下のプロンプトを実行させる:

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
| `database does not exist` | DB名の誤り | `\l`（PG）/ `SHOW DATABASES`（MySQL）で確認 |
| Oracle `ORA-12154` | TNS 解決失敗 | Instant Client の `tnsnames.ora` 確認 |
| MongoDB `AuthenticationFailed` | authSource 不一致 | `?authSource=admin` の有無確認 |
| Redis `NOAUTH` | パスワード設定済みなのに未指定 | `REDIS_PASSWORD` を設定 |
| `npx: command not found` | Node.js 未インストール | Node.js 18+ をインストール |

---

## Phase 3: 実際の使い方例

接続が確認できたら、以下のプロンプトをそのまま使える。

### スキーマ把握
```
usersテーブルのカラム定義とインデックスをすべて教えてください。
```

### スロークエリ診断
```
以下のクエリの実行計画を取得して、改善案を提示してください。
SELECT * FROM orders WHERE customer_id = 123 ORDER BY created_at DESC;
```

### データ調査
```
2026年4月の新規注文件数と、前月比を集計してください。
```

### MongoDB 集計
```
statusが"active"のドキュメント数をcollectionごとに集計してください。
```

### Redis 調査
```
"session:"で始まるキーの件数と、TTLが1時間以内に切れるキーを一覧表示してください。
```

### 複数 DB 横断（PostgreSQL + Redis の例）
```
PostgreSQLのsessionsテーブルとRedisのsession:*キーを照合して、
DBには存在するがRedisに存在しないセッションIDを抽出してください。
```

---

## .gitignore への追加（必須）

`.mcp.json` にパスワードを直書きした場合は必ず追加すること:

```
# .gitignore
.mcp.json
.claude.json
```

パスワードを `env` で分離している場合は `.env` を追加:

```
.env
.env.local
```

---

## Forbidden (D4/D5)

- `.mcp.json` に接続文字列を貼った後、「セキュリティに注意が必要です」のような当たり前の警告を長々と書かない。`.gitignore` に追加するよう1行で指示する。
- 「接続できました」「正常に動作しています」など状態を繰り返し言わない。次のアクション（クエリ例）に即進む。
- DB が起動していなかった場合、原因推測を並べない。確認コマンドを1つ示す。
