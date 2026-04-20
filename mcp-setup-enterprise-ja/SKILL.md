---
name: mcp-setup-enterprise-ja
description: Sets up MCP server for Japanese enterprise databases (Oracle 19c / SQL Server) on Windows PCs. Handles Oracle Instant Client installation, Windows PATH configuration, corporate firewall issues, and remote DB connections. Use when a Japanese SI developer wants to connect Claude Code to their internal Oracle or SQL Server database.
---

# 社内 DB MCP セットアップ（Oracle 19c / SQL Server）

## Activation

このスキルが呼ばれたら、以下のフェーズを順番に実行してください。フェーズをスキップしてはなりません。

---

## Phase 0: DB 種別・環境確認

ユーザーに以下を確認してください（すでにコンテキストが提供されている場合はスキップ）:

> 以下を教えてください。
>
> **DB 種別**: Oracle 19c / SQL Server / 両方
>
> **OS**: Windows（このスキルは Windows 前提です）
>
> **接続方式**:
> - Oracle: IP アドレスまたはホスト名・ポート・SERVICE_NAME または SID・ユーザー名
> - SQL Server: IP アドレスまたはホスト名・インスタンス名（名前付きの場合）・認証方式（SQL認証 or Windows認証）
>
> **社内環境の状況**:
> - プロキシサーバーはありますか？（yes/no）
> - IT 部門への申請は済んでいますか？（1521/TCP or 1433/TCP のポート開放）

---

## Phase 1: 前提条件チェック

### Node.js 確認

```powershell
node -v
# v18.x.x 以上が表示されればOK
```

Node.js が入っていない場合は https://nodejs.org から LTS 版をインストールするよう案内してください。

### Oracle Instant Client 確認（Oracle の場合のみ）

```powershell
# oracle ディレクトリの存在確認
Test-Path "C:\oracle"
```

Instant Client が入っていない場合は以下の手順を案内してください:

1. https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html から Basic Package をダウンロード
2. `C:\oracle\instantclient_21_x` に展開（パスにスペースを含めないこと）
3. システム環境変数 PATH に `C:\oracle\instantclient_21_x` を追加
4. ターミナルを再起動

### ポート疎通確認

```powershell
# Oracle
Test-NetConnection -ComputerName [DBサーバーIP] -Port 1521

# SQL Server
Test-NetConnection -ComputerName [DBサーバーIP] -Port 1433

# TcpTestSucceeded : True → 接続可能
# TcpTestSucceeded : False → IT部門にポート開放を申請
```

---

## Phase 2: .mcp.json 生成

ユーザーの入力情報に基づいて `.mcp.json` を生成してください。

### Oracle パターン

```json
{
  "mcpServers": {
    "oracle": {
      "command": "npx",
      "args": ["-y", "oracle-mcp-server"],
      "env": {
        "ORACLE_USER": "YOUR_USER",
        "ORACLE_PASSWORD": "YOUR_PASSWORD",
        "ORACLE_CONNECTION_STRING": "192.168.x.x:1521/SERVICE_NAME"
      }
    }
  }
}
```

**接続文字列パターン:**
- SERVICE_NAME（推奨）: `192.168.x.x:1521/SERVICE_NAME`
- SID（旧来）: `(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=IP)(PORT=1521))(CONNECT_DATA=(SID=ORCL)))`
- ホスト名: `dbserver.company.local:1521/SERVICE_NAME`

TNS は不要。Easy Connect 形式で直接接続できます。

### SQL Server パターン（SQL 認証）

```json
{
  "mcpServers": {
    "sqlserver": {
      "command": "npx",
      "args": ["-y", "mssql-mcp-server"],
      "env": {
        "MSSQL_HOST": "192.168.x.x",
        "MSSQL_PORT": "1433",
        "MSSQL_USER": "dev_readonly",
        "MSSQL_PASSWORD": "YOUR_PASSWORD",
        "MSSQL_DATABASE": "YOUR_DB",
        "MSSQL_ENCRYPT": "false",
        "MSSQL_TRUST_SERVER_CERTIFICATE": "true"
      }
    }
  }
}
```

### SQL Server パターン（Windows 認証・推奨）

```json
{
  "mcpServers": {
    "sqlserver": {
      "command": "npx",
      "args": ["-y", "mssql-mcp-server"],
      "env": {
        "MSSQL_HOST": "192.168.x.x",
        "MSSQL_PORT": "1433",
        "MSSQL_DATABASE": "YOUR_DB",
        "MSSQL_TRUSTED_CONNECTION": "true",
        "MSSQL_ENCRYPT": "false",
        "MSSQL_TRUST_SERVER_CERTIFICATE": "true"
      }
    }
  }
}
```

### 名前付きインスタンスの場合（SQL Server）

`MSSQL_HOST` にバックスラッシュでインスタンス名を付加します（JSON では `\\` でエスケープ）:

```json
"MSSQL_HOST": "192.168.x.x\\SQLEXPRESS"
```

### プロキシが必要な場合（両方共通）

`env` に追加してください:

```json
"HTTPS_PROXY": "http://proxy.company.local:8080",
"HTTP_PROXY": "http://proxy.company.local:8080"
```

---

## Phase 3: 接続テスト・エラー対処

`.mcp.json` 保存後、Claude Code を再起動してから接続を確認してください。

### 接続確認プロンプト

```
利用可能な MCP ツールを一覧表示してください
```

または:

```
テーブル一覧を表示してください
```

### エラー別対処フロー

**Oracle エラー:**

| エラー | 対処 |
|---|---|
| `ORA-12154` | 接続文字列の形式を確認。Easy Connect 形式（`host:port/service`）を使う |
| `ORA-01017` | ユーザー名・パスワードを確認。大文字小文字に注意 |
| `ORA-12541` | リスナーが未起動かポートが違う。DBA に確認 |
| `ORA-12170` | ファイアウォールで 1521 がブロックされている。IT 部門に申請 |
| `NJS-045` | Oracle Instant Client が見つからない。PATH を確認 |

**SQL Server エラー:**

| エラー | 対処 |
|---|---|
| `Login failed for user` | ユーザー名・パスワードを確認。SQL 認証が有効か DBA に確認 |
| `Cannot connect to server` | ホスト名・ポートを確認。`Test-NetConnection` で疎通チェック |
| `SSL routines: wrong version number` | `MSSQL_TRUST_SERVER_CERTIFICATE: "true"` を追加 |
| `The server was not found` | インスタンス名を確認。SQL Server Browser サービスが起動しているか確認 |

---

## Phase 4: セキュリティ設定

### 読み取り専用ユーザー作成 SQL

DBA に依頼するための SQL を生成してください。

**Oracle:**

```sql
CREATE USER mcp_readonly IDENTIFIED BY "StrongPassword123!";
GRANT CREATE SESSION TO mcp_readonly;
-- 特定スキーマのみ（推奨）
GRANT SELECT ON APP_SCHEMA.ORDERS TO mcp_readonly;
GRANT SELECT ON APP_SCHEMA.CUSTOMERS TO mcp_readonly;
```

**SQL Server:**

```sql
-- SQL認証の場合
CREATE LOGIN mcp_readonly WITH PASSWORD = 'StrongPassword123!';
USE [YOUR_DB];
CREATE USER mcp_readonly FOR LOGIN mcp_readonly;
EXEC sp_addrolemember 'db_datareader', 'mcp_readonly';
```

### .gitignore の確認・追加

```
.mcp.json
.env
.env.local
```

`.mcp.json` がすでに git に追加されていないか確認:

```powershell
git ls-files .mcp.json
# 何も表示されなければOK（未追加の状態）
```

---

## Forbidden — 禁止事項

- **本番 DB への接続禁止**: 開発・検証環境のみで使用する
- **パスワードの git コミット禁止**: `.mcp.json` と `.env` は必ず `.gitignore` に追加する
- **`sa` ユーザーや DBA 権限での接続禁止**: 読み取り専用ユーザーを使う
- **顧客の個人情報・機密データを AI チャットに貼り付けない**
