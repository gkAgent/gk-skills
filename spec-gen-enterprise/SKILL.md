---
name: spec-gen-enterprise
description: Generates structured design specification documents from existing source code for enterprise systems. Supports Java × Oracle/PostgreSQL/MySQL/SQL Server, TypeScript × all DBs, C#/.NET × all DBs, VB.NET × all DBs, COBOL × all DBs. Use when a user wants to generate a spec document from legacy source code, document an undocumented system, create detailed design specs from existing implementation, or produce design documents for a maintenance project. Follows the 5 D-principles: question-driven start, coverage visualization, Files>Prompts, 80%-NOT instructions, and no AI-voice filler.
---

# Spec Generation Kit — Enterprise Edition

## Activation

When this skill is invoked, follow the exact flow below. Do not skip phases.

---

## Phase 0: Identify Language × DB

Ask the user:

> Please tell me the language and database combination for this system.
>
> **Language**: Java / TypeScript / C# (.NET) / VB.NET / COBOL / Other
> **DB**: Oracle / PostgreSQL / MySQL / SQL Server / SQLite / No DB / Other

If the user has already provided this context, skip to Phase 1.

---

## Phase 1: Question-Driven Setup (D1)

**Never start generating the spec without first completing this phase.**

Ask in three stages. Wait for answers before proceeding.

### Stage 1 — System basics
1. What is the official system name?
2. Which features, screens, or batch jobs are in scope? (multiple allowed)
3. Have you attached the source code, DDL, and/or configuration files?

### Stage 2 — Tech stack (language-specific)

**Java users:**
- Java version (8 / 11 / 17 / 21)
- Framework (Spring Boot / Spring MVC / Jakarta EE / Plain Java)
- Data access method (JPA/Hibernate / MyBatis / JDBC Template / jOOQ / PL/SQL calls)
- Oracle only: version, presence of PL/SQL Packages

**TypeScript users:**
- Framework (React / Next.js / Vue / Angular / Express / NestJS)
- ORM/client (Prisma / TypeORM / Drizzle / Knex / raw SQL)

**C# / VB.NET users:**
- .NET version
- Project type (WinForms / WPF / ASP.NET Core MVC / Web API / WebForms)
- Data access (Entity Framework Core / Dapper / ADO.NET / Stored Procedures)

**COBOL users:**
- Runtime (IBM Enterprise COBOL / Micro Focus / OpenCOBOL)
- DB access method (EXEC SQL / VSAM / DB2 / IMS)

### Stage 3 — Output preferences
- Granularity: per screen / per feature / per batch / per API / per DB
- Output format: Markdown / follow a provided template
- Are there any high-priority features to address first?

After collecting answers, **construct the `[INPUT]` block and show it to the user for confirmation** before generating.

---

## Phase 2: Coverage Plan (D2)

Before writing any spec, output a **Coverage Plan**:

```
## Coverage Plan

| # | Target | Type | Status |
|---|--------|------|--------|
| 1 | [Feature name] | Screen / Batch / API | [ ] Not started |
| 2 | ... | ... | [ ] Not started |
```

Rules:
- List ALL targets identified from the provided source code or user input
- Do NOT choose which ones to process based on your own judgment
- After each item, update the status to `[x] Done` or `[△] Has TODOs`
- Never stop processing mid-list unless the user explicitly says so

---

## Phase 3: Spec Generation (per item)

For each item in the coverage plan, generate a spec following this structure:

```markdown
# Design Specification — [Feature Name]

## 0. Basic Information
| Field | Value |
|---|---|
| System Name | |
| Feature Name | |
| Target Files | |
| Language / FW / DB | |
| Date | |

## 1. Processing Overview
[1–3 sentences, concise]

## 2. Inputs
### 2.1 Parameters / Screen Inputs
| Field Name | Type | Required | Description |
|---|---|---|---|

## 3. Processing Flow
[Numbered steps. Only describe what is evidenced in the source]

## 4. DB Operations
| Type | Table / View | Condition | Notes |
|---|---|---|---|

## 5. Output / Return Values

## 6. Error Handling

## 7. TODOs / Items Requiring Confirmation
[List all items lacking sufficient evidence. Do not leave this empty when uncertainty exists]
```

---

## Forbidden — Absolute Prohibitions (D4)

These rules take priority over everything else:

- **No speculative assertions**: Do not state anything as fact if it has no basis in the source code. Phrases like "it is assumed that…" or "generally speaking…" are also prohibited.
- **No mid-list stops**: Do not stop before completing the full coverage plan.
- **No AI voice (D5)**: Do not write verbose introductory sentences like "This system is designed to…" or "It is important to…".
- **No prompts over 100 characters (D3)**: Do not ask the user to write long prompts.
- **No hiding TODOs**: Unresolved items must go into Section 7. Do not omit them to make the spec look cleaner.

---

## Notes for specific language × DB combinations

### Java × Oracle
- When PL/SQL Package calls exist, include the Package name, Procedure name, and IN/OUT arguments in Section 4 (DB Operations)
- Always explicitly document how `SYS_REFCURSOR` is handled
- Only mention Oracle-version-specific features (Partitioning, Advanced Queuing, etc.) when there is source-level evidence

### TypeScript × any DB
- Document the `async/await` error-handling pattern explicitly
- For Prisma: reference the schema.prisma model definitions as the source of truth

### VB.NET × SQL Server
- Note whether stored procedure calls use `CommandType.StoredProcedure`
- Document transaction management approach (`SqlTransaction` vs `TransactionScope`)
- Note whether `Option Strict On` is declared — if absent, flag as a concern
- `OracleCommand.Parameters` with `BindByName = True` (VB.NET × Oracle): verify that named binding is consistent across all commands

### C# × SQL Server / Oracle
- Document `using` blocks for `SqlConnection` / `OracleConnection` to confirm resource cleanup
- For stored procedures, list all `SqlParameter` / `OracleParameter` entries with direction (Input / Output / InputOutput)

### COBOL × DB2
- Include a `SQLCODE` check after every `EXEC SQL` block
- Include the WORKING-STORAGE HOST variable inventory in the spec
- Verify `CURSOR` OPEN / FETCH / CLOSE symmetry
