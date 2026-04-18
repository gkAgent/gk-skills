# gk-skills — Enterprise Claude Code Skills for Japanese SI Teams

> **39 skills** covering the full Japanese SI project lifecycle.
> Built from real projects: 300k-line VB.NET migrations, Oracle 19c systems, COBOL batch modernization.

---

## Quick Start

```bash
# 1. Install a skill (one-time per skill)
npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja

# 2. Open Claude Code in your project
cd /path/to/your-project
claude

# 3. Invoke the skill
/spec-gen-enterprise-ja
```

Claude asks clarifying questions first (Phase 0), then runs to completion — no mid-task stops.

---

## Which Skill Do I Need?

| Situation | Skill to use |
|---|---|
| **コードから仕様書を書きたい** | `spec-gen-enterprise-ja` (全言語) or 言語×DB 個別版 |
| **本番リリース前にコードをレビューしたい** | `code-review-[java\|csharp\|typescript\|vbnet\|cobol]` |
| **セキュリティ診断をしたい (OWASP)** | `security-review-web` |
| **スロークエリ / N+1 を直したい** | `perf-review-sql` |
| **VB.NET → TypeScript に移行したい** | `migration-vbnet-typescript` |
| **C# → TypeScript に移行したい** | `migration-csharp-typescript` |
| **VB.NET → ASP.NET Core に移行したい（.NET内）** | `migration-vbnet-aspnetcore` |
| **COBOL → TypeScript/Node.js に移行したい** | `migration-cobol-typescript` |
| **Oracle → PostgreSQL に移行したい** | `db-migration-oracle-postgresql` |
| **移行せずコードを整理したい** | `refactor-enterprise-ja` |
| **テスト仕様書とテストコードを生成したい** | `test-gen-enterprise-ja` |
| **OpenAPI 仕様書を既存コードから生成したい** | `api-spec-openapi` |
| **新規参画者向けドキュメントを作りたい** | `onboarding-doc-ja` |
| **障害報告書・RCA を書きたい** | `incident-rca-ja` |
| **仕様書の TODO を解消したい** | `todo-resolution-ja` |
| **DBにClaudeから直接クエリしたい** | `mcp-setup-db` |
| **GitHub をClaudeから操作したい (Issues/PR/Actions)** | `mcp-setup-github` |
| **監視ツール (Datadog/Sentry) をClaudeから確認したい** | `mcp-setup-monitoring` |
| **Cloudflare Workers/D1/R2/KV をClaudeから操作したい** | `mcp-setup-cloudflare` |
| **Slack/LINE/Discord の通知をClaudeから送りたい** | `mcp-setup-communication` |
| **既存の .mcp.json のセキュリティを診断したい** | `mcp-audit` |

---

## Skills (33)

### Spec Generation (詳細仕様書) — 14 skills

> **迷ったら `spec-gen-enterprise-ja`。** 特定の言語×DBに深い内容が必要な場合は個別版。

| Skill | Stack | Install |
|---|---|---|
| [spec-gen-enterprise-ja](./spec-gen-enterprise-ja/SKILL.md) | Java × Oracle/PG/MySQL/MSSQL + TypeScript + C# + VB.NET + COBOL | `npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja` |
| [spec-gen-java-oracle](./spec-gen-java-oracle/SKILL.md) | Java × Oracle (MyBatis/JPA, ROWNUM/NUMBER/BindByName) | `npx skills@latest add gkAgent/gk-skills/spec-gen-java-oracle` |
| [spec-gen-java-postgresql](./spec-gen-java-postgresql/SKILL.md) | Java × PostgreSQL (Spring Data JPA, JSONB/UUID/N+1) | `npx skills@latest add gkAgent/gk-skills/spec-gen-java-postgresql` |
| [spec-gen-java-mysql](./spec-gen-java-mysql/SKILL.md) | Java × MySQL (utf8mb4/TINYINT(1)/ON DUPLICATE KEY) | `npx skills@latest add gkAgent/gk-skills/spec-gen-java-mysql` |
| [spec-gen-java-sqlserver](./spec-gen-java-sqlserver/SKILL.md) | Java × SQL Server (MERGE/IDENTITY/NVARCHAR/Azure AD) | `npx skills@latest add gkAgent/gk-skills/spec-gen-java-sqlserver` |
| [spec-gen-typescript-postgresql](./spec-gen-typescript-postgresql/SKILL.md) | TypeScript × PostgreSQL (Prisma/Drizzle/pg) | `npx skills@latest add gkAgent/gk-skills/spec-gen-typescript-postgresql` |
| [spec-gen-typescript-mysql](./spec-gen-typescript-mysql/SKILL.md) | TypeScript × MySQL / PlanetScale | `npx skills@latest add gkAgent/gk-skills/spec-gen-typescript-mysql` |
| [spec-gen-typescript-sqlserver](./spec-gen-typescript-sqlserver/SKILL.md) | TypeScript × SQL Server (Prisma/mssql, Azure SQL) | `npx skills@latest add gkAgent/gk-skills/spec-gen-typescript-sqlserver` |
| [spec-gen-csharp-sqlserver](./spec-gen-csharp-sqlserver/SKILL.md) | C# × SQL Server (ASP.NET Core/WinForms/WPF) | `npx skills@latest add gkAgent/gk-skills/spec-gen-csharp-sqlserver` |
| [spec-gen-csharp-oracle](./spec-gen-csharp-oracle/SKILL.md) | C# × Oracle (ODP.NET BindByName/NUMBER→decimal) | `npx skills@latest add gkAgent/gk-skills/spec-gen-csharp-oracle` |
| [spec-gen-csharp-postgresql](./spec-gen-csharp-postgresql/SKILL.md) | C# × PostgreSQL (EF Core Npgsql, JSONB/UUID/arrays) | `npx skills@latest add gkAgent/gk-skills/spec-gen-csharp-postgresql` |
| [spec-gen-vbnet-sqlserver](./spec-gen-vbnet-sqlserver/SKILL.md) | VB.NET × SQL Server (WinForms/WebForms/WPF) | `npx skills@latest add gkAgent/gk-skills/spec-gen-vbnet-sqlserver` |
| [spec-gen-vbnet-oracle](./spec-gen-vbnet-oracle/SKILL.md) | VB.NET × Oracle (ODP.NET/DataAdapter, レガシー) | `npx skills@latest add gkAgent/gk-skills/spec-gen-vbnet-oracle` |
| [spec-gen-cobol-db2](./spec-gen-cobol-db2/SKILL.md) | COBOL × IBM DB2 (z/OS / LUW / CICS) | `npx skills@latest add gkAgent/gk-skills/spec-gen-cobol-db2` |

### Code Review — 6 skills

> **迷ったら `code-review-enterprise-ja`。** 言語固有の深いチェックは個別版。

| Skill | Focus | Install |
|---|---|---|
| [code-review-enterprise-ja](./code-review-enterprise-ja/SKILL.md) | Java + TypeScript + C# + VB.NET + COBOL | `npx skills@latest add gkAgent/gk-skills/code-review-enterprise-ja` |
| [code-review-java](./code-review-java/SKILL.md) | @Transactional自己呼び出し / N+1 / MyBatis `${}` / Modernizationスコア | `npx skills@latest add gkAgent/gk-skills/code-review-java` |
| [code-review-csharp](./code-review-csharp/SKILL.md) | async void / HttpClient都度new / EF Core N+1 / [Authorize]漏れ | `npx skills@latest add gkAgent/gk-skills/code-review-csharp` |
| [code-review-typescript](./code-review-typescript/SKILL.md) | any/unknown/as / dangerouslySetInnerHTML / env var 露出 | `npx skills@latest add gkAgent/gk-skills/code-review-typescript` |
| [code-review-vbnet](./code-review-vbnet/SKILL.md) | On Error Resume Next / Option Strict Off / Modernizationスコア | `npx skills@latest add gkAgent/gk-skills/code-review-vbnet` |
| [code-review-cobol](./code-review-cobol/SKILL.md) | SQLCODE完全性 / CURSOR対称性 / CICS SYNCPOINT | `npx skills@latest add gkAgent/gk-skills/code-review-cobol` |

### Migration — 5 skills

| Skill | Path | Install |
|---|---|---|
| [migration-vbnet-typescript](./migration-vbnet-typescript/SKILL.md) | VB.NET → TypeScript/React (5フェーズ) | `npx skills@latest add gkAgent/gk-skills/migration-vbnet-typescript` |
| [migration-csharp-typescript](./migration-csharp-typescript/SKILL.md) | C# → TypeScript/Next.js/NestJS | `npx skills@latest add gkAgent/gk-skills/migration-csharp-typescript` |
| [migration-vbnet-aspnetcore](./migration-vbnet-aspnetcore/SKILL.md) | VB.NET → ASP.NET Core (.NET内、低リスク) | `npx skills@latest add gkAgent/gk-skills/migration-vbnet-aspnetcore` |
| [migration-cobol-typescript](./migration-cobol-typescript/SKILL.md) | COBOL → TypeScript/Node.js (COMP-3/JCL対応) | `npx skills@latest add gkAgent/gk-skills/migration-cobol-typescript` |
| [db-migration-oracle-postgresql](./db-migration-oracle-postgresql/SKILL.md) | Oracle → PostgreSQL (ROWNUM/PL/SQL/型変換完全対応) | `npx skills@latest add gkAgent/gk-skills/db-migration-oracle-postgresql` |

### Quality & Security — 3 skills

| Skill | Focus | Install |
|---|---|---|
| [security-review-web](./security-review-web/SKILL.md) | OWASP Top 10 (2021) 全項目 — IDOR/SQLi/JWT/SSRF | `npx skills@latest add gkAgent/gk-skills/security-review-web` |
| [perf-review-sql](./perf-review-sql/SKILL.md) | N+1 / インデックス設計 / 実行計画 (Oracle/PG/MySQL/MSSQL) | `npx skills@latest add gkAgent/gk-skills/perf-review-sql` |
| [refactor-enterprise-ja](./refactor-enterprise-ja/SKILL.md) | Fat イベントハンドラ→サービス層 / On Error GoTo→Try/Catch | `npx skills@latest add gkAgent/gk-skills/refactor-enterprise-ja` |

### Testing — 1 skill

| Skill | Focus | Install |
|---|---|---|
| [test-gen-enterprise-ja](./test-gen-enterprise-ja/SKILL.md) | テスト仕様書 + JUnit/Jest/xUnit コード生成 (同値分割/境界値) | `npx skills@latest add gkAgent/gk-skills/test-gen-enterprise-ja` |

### API & Documentation — 3 skills

| Skill | Focus | Install |
|---|---|---|
| [api-spec-openapi](./api-spec-openapi/SKILL.md) | 既存コードから OpenAPI 3.0 YAML 生成 | `npx skills@latest add gkAgent/gk-skills/api-spec-openapi` |
| [onboarding-doc-ja](./onboarding-doc-ja/SKILL.md) | 新規参画者向けシステム概要書・環境構築手順 | `npx skills@latest add gkAgent/gk-skills/onboarding-doc-ja` |
| [incident-rca-ja](./incident-rca-ja/SKILL.md) | 障害報告書・5 Why・顧客報告メール・再発防止計画 | `npx skills@latest add gkAgent/gk-skills/incident-rca-ja` |

### Productivity — 1 skill

| Skill | Focus | Install |
|---|---|---|
| [todo-resolution-ja](./todo-resolution-ja/SKILL.md) | 仕様書に溜まった TODO を2パスで解消 | `npx skills@latest add gkAgent/gk-skills/todo-resolution-ja` |

### MCP Setup & Audit — 6 skills

> Claude Code から外部サービスを直接操作するための MCP サーバー設定を自動化する。

| Skill | Focus | Install |
|---|---|---|
| [mcp-setup-db](./mcp-setup-db/SKILL.md) | PostgreSQL / MySQL / SQLite の .mcp.json 生成・接続テスト | `npx skills@latest add gkAgent/gk-skills/mcp-setup-db` |
| [mcp-setup-github](./mcp-setup-github/SKILL.md) | GitHub MCP — PAT取得・Issues/PR/Actions 操作例 | `npx skills@latest add gkAgent/gk-skills/mcp-setup-github` |
| [mcp-setup-monitoring](./mcp-setup-monitoring/SKILL.md) | Datadog / Sentry MCP — ログ取得・アラート確認 | `npx skills@latest add gkAgent/gk-skills/mcp-setup-monitoring` |
| [mcp-setup-cloudflare](./mcp-setup-cloudflare/SKILL.md) | Cloudflare Workers / D1 / R2 / KV MCP 設定 | `npx skills@latest add gkAgent/gk-skills/mcp-setup-cloudflare` |
| [mcp-setup-communication](./mcp-setup-communication/SKILL.md) | Slack / LINE / Discord MCP — Bot設定・メッセージ送受信 | `npx skills@latest add gkAgent/gk-skills/mcp-setup-communication` |
| [mcp-audit](./mcp-audit/SKILL.md) | .mcp.json / .claude.json のセキュリティ診断・レポート出力 | `npx skills@latest add gkAgent/gk-skills/mcp-audit` |

---

## How to Use

### Step 1: Claude Code をインストール（未インストールの場合）

```bash
npm install -g @anthropic-ai/claude-code
```

### Step 2: スキルをインストール

```bash
# 1つだけ試す場合
npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja

# 全部まとめてインストール
npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja
npx skills@latest add gkAgent/gk-skills/code-review-enterprise-ja
npx skills@latest add gkAgent/gk-skills/security-review-web
npx skills@latest add gkAgent/gk-skills/test-gen-enterprise-ja
npx skills@latest add gkAgent/gk-skills/todo-resolution-ja
```

### Step 3: プロジェクトで Claude Code を起動してスキルを呼び出す

```bash
cd /path/to/your-java-project
claude
```

```
# Claude Code のプロンプトで
/spec-gen-enterprise-ja
```

スキルが起動すると、対象システムの確認（Phase 0）から始まります。
コードを貼るか、ファイルを `@` で参照するだけで出力が生成されます。

### 動作例（spec-gen-enterprise-ja）

```
> /spec-gen-enterprise-ja

移行対象を教えてください。
現行: Java 17 / Spring Boot 3 / MyBatis 3
DB: Oracle 19c
規模: 80クラス / 40 Mapper ファイル

> 上記の通りです。対象は src/main/java/com/example/service/EmployeeService.java です。

## EmployeeService — 詳細仕様書

### クラス概要
...（以降、完全な仕様書が生成される）
```

---

## Install Packs

### はじめての1本（迷ったらこれ）
```bash
npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja
```

### SI標準セット（spec + review + test + todo）
```bash
npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja
npx skills@latest add gkAgent/gk-skills/code-review-enterprise-ja
npx skills@latest add gkAgent/gk-skills/test-gen-enterprise-ja
npx skills@latest add gkAgent/gk-skills/todo-resolution-ja
```

### セキュリティ・品質セット
```bash
npx skills@latest add gkAgent/gk-skills/security-review-web
npx skills@latest add gkAgent/gk-skills/perf-review-sql
npx skills@latest add gkAgent/gk-skills/code-review-enterprise-ja
```

### 移行プロジェクトセット（VB.NET案件）
```bash
npx skills@latest add gkAgent/gk-skills/migration-vbnet-typescript
npx skills@latest add gkAgent/gk-skills/code-review-vbnet
npx skills@latest add gkAgent/gk-skills/refactor-enterprise-ja
```

### Oracle → PostgreSQL 移行セット
```bash
npx skills@latest add gkAgent/gk-skills/db-migration-oracle-postgresql
npx skills@latest add gkAgent/gk-skills/spec-gen-java-oracle
npx skills@latest add gkAgent/gk-skills/spec-gen-java-postgresql
```

---

## Why These Exist

Standard Claude prompts have 3 failure modes on enterprise projects:

1. **Forget context** — start over every session
2. **Stop mid-task** — "I've covered the main points" after 30% of the code
3. **Repeat mistakes** — same wrong pattern every time

Every skill in this repo enforces 5 rules that fix all three:

| Rule | What it fixes |
|---|---|
| **D1 — Question-driven start** | Claude generating specs before understanding the system |
| **D2 — Full coverage, no self-stopping** | Claude deciding which files to skip |
| **D3 — Files > Prompts** | Writing 200-character prompts every session |
| **D4 — 80% NOT instructions** | Vague positive instructions Claude interprets loosely |
| **D5 — No AI voice** | "It is important to note that..." filler in every output |

---

## Background

Built by [gkAgent](https://github.com/gkAgent) — a solo developer running a 14-member virtual AI company.

Extracted from real Japanese SI projects: a 300k-line VB.NET × Oracle system, COBOL batch modernization, Spring Boot × Oracle 19c codebases.

These skills are part of the [AI Agent Development Guide](https://aidev-guide.pages.dev/) — a comprehensive template collection and methodology for enterprise AI development in Japan (Japanese, paid).

If these skills saved you time, a ⭐ on this repo helps others find it.

Issues and PRs welcome.
