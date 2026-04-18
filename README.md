# gk-skills — Enterprise Claude Code Skills for Japanese SI Teams

> Install any skill in one command:
> ```
> npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja
> ```

Claude Code Skills built from real Japanese SI projects:
300k-line VB.NET migrations, Oracle 19c systems, COBOL batch modernization.

**Why these exist:** Standard Claude prompts forget context, repeat mistakes, and stop mid-task.
These skills enforce 5 rules that fix all three — permanently.

---

## Skills

### [spec-gen-enterprise-ja](./spec-gen-enterprise-ja/SKILL.md) — Spec Document Generation

Generate 詳細仕様書 (design specs) from existing source code.

**Languages × DBs:** Java × Oracle/PostgreSQL/MySQL/SQL Server · TypeScript × all · C#/.NET · VB.NET · COBOL

**Use when:** you need to document a legacy system, generate a spec from undocumented code, or produce 詳細仕様書 for a maintenance project.

```bash
npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja
```

---

### [code-review-enterprise-ja](./code-review-enterprise-ja/SKILL.md) — Structured Code Review

Language-specific code review with security-first checklists. Definitive output — not "this might be an issue."

**Languages:** Java (8/11/17/21) · TypeScript/React · C# (.NET) · VB.NET · COBOL

```bash
npx skills@latest add gkAgent/gk-skills/code-review-enterprise-ja
```

---

### [migration-vbnet-typescript](./migration-vbnet-typescript/SKILL.md) — VB.NET → TypeScript Migration

5-phase migration methodology. Risk assessment, code conversion patterns, stakeholder summary.

**Covers:** WinForms · WebForms · WPF · Oracle/SQL Server → PostgreSQL/MySQL

```bash
npx skills@latest add gkAgent/gk-skills/migration-vbnet-typescript
```

---

### [todo-resolution-ja](./todo-resolution-ja/SKILL.md) — TODO Resolution (2-Pass)

Clear accumulated TODOs from spec documents. Extract → categorize → generate questions → apply diff update.

**Works with:** any language, any domain

```bash
npx skills@latest add gkAgent/gk-skills/todo-resolution-ja
```

---

## Install All 4 Skills

```bash
npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja
npx skills@latest add gkAgent/gk-skills/code-review-enterprise-ja
npx skills@latest add gkAgent/gk-skills/migration-vbnet-typescript
npx skills@latest add gkAgent/gk-skills/todo-resolution-ja
```

---

## The 5 Rules Behind Every Skill

Every skill in this repo enforces the same 5 principles. They come from watching Claude fail repeatedly on real enterprise projects.

| Rule | What it fixes |
|---|---|
| **D1 — Question-driven start** | Claude generating specs before knowing the system |
| **D2 — Full coverage, no self-stopping** | Claude deciding which files to skip |
| **D3 — Files > Prompts** | Users writing 200-character prompts every session |
| **D4 — 80% NOT instructions** | Vague positive instructions that Claude interprets loosely |
| **D5 — No AI voice** | "It is important to note that..." filler in output |

---

## Background

Built by [gkAgent](https://github.com/gkAgent) — a solo developer running a 14-member virtual AI company.

These skills are extracted from the [AI Agent Development Guide](https://github.com/gkAgent/aidev-guide) — a 560+ file template collection for enterprise AI development in Japan.

Issues and PRs welcome.
