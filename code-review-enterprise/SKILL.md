---
name: code-review-enterprise
description: Performs structured code review for enterprise systems using language-specific checklists. Supports Java (8/11/17/21), TypeScript, C#/.NET, VB.NET, COBOL. Use when a user wants to review source code before deployment, audit legacy code for quality or security, review a pull request, conduct a pre-migration code assessment, or generate a review report in Markdown. Follows the 5 D-principles: question-driven start, full-coverage traversal, Files>Prompts, 80%-NOT instructions, no AI-voice filler.
---

# Code Review Kit — Enterprise Edition

## Activation

When this skill is invoked, follow the exact flow below.

---

## Phase 0: Identify Target

Ask the user:

> Please tell me what you want to review.
>
> - **Language**: Java / TypeScript / C# / VB.NET / COBOL / Other
> - **Target**: File name(s), feature name, or PR number
> - **Focus**: Security / Quality / Performance / All of the above

---

## Phase 1: Coverage Plan (D2)

List ALL files or functions to review before starting. Update status after each.

```
## Review Coverage Plan

| # | File / Function | Focus | Status |
|---|---|---|---|
| 1 | [file name] | General | [ ] Not started |
```

Do not skip files. Do not prioritize by your own judgment.

---

## Phase 2: Language-Specific Review

### Java Review Checklist

**A. Version & Dependencies**
- Does the Java version (8/11/17/21) match `sourceCompatibility` in `pom.xml` / `build.gradle`?
- Are Spring Boot 2.7 or older (end-of-life) or Log4j 1.x / 2.x (< 2.17.1) present?
- Are there raw-type or unchecked warnings that would appear under `-Xlint:all`?

**B. Null Safety**
- Are null values from external inputs and DB return values handled?
- Is `Optional` used as a field or parameter type (should be return-value only)?
- Is `Optional.get()` called without a preceding `isPresent()` check?

**C. Exception Handling**
- Is `catch(Exception e)` silently swallowing exceptions?
- Is checked-exception chaining used correctly (`throw new XxxException("msg", e)`)?
- Is the relationship between transaction boundaries and exceptions correct (`@Transactional(rollbackFor=)`)?

**D. Security**
- SQL injection: no string-concatenated SQL; `PreparedStatement` / `NamedParameterJdbcTemplate` is mandatory
- XSS: Thymeleaf `th:text` vs `th:utext`; JSP `<c:out>` usage
- CSRF protection: is Spring Security's `csrf()` enabled?
- Authentication / authorization gaps: `@PreAuthorize`, `SecurityContextHolder` usage

**E. Oracle-Specific (when applicable)**
- `CallableStatement` for Package calls: are all IN/OUT parameter bindings present?
- Is `SYS_REFCURSOR` closed after use?
- Are `OracleConnection` instances leaking?

---

### TypeScript Review Checklist

**A. Type Safety**
- Where is the `any` type used? (Flag as candidates for `unknown`)
- Are `as` casts overused? (Consider replacing with type guard functions)
- Is the `!` (non-null assertion) operator necessary in each case?

**B. Async Handling**
- Is there a corresponding `try/catch` for every `async/await` block?
- Are `.catch()` handlers missing from Promise chains?
- Is error handling present when using `Promise.all`?

**C. Security**
- Is `dangerouslySetInnerHTML` used? (XSS risk)
- Is `process.env` referenced directly on the client side? (Server-side only is acceptable)
- SQL injection: are raw query strings in ORMs built via concatenation?

**D. React-Specific (when applicable)**
- Are `useEffect` dependency arrays complete?
- Are components oversized? (Over 200 lines is a split candidate)
- Is `index` used as a `key` prop where reordering is possible? (Flag as issue)

---

### C# / VB.NET Review Checklist

**A. Resource Management**
- Are `IDisposable` implementations wrapped in `using` blocks?
- Are `SqlConnection` / `SqlCommand` instances leaking?

**B. Exception Handling**
- Is `catch(Exception)` swallowing stack traces?
- Is `throw ex` used instead of `throw`? (`throw ex` destroys the original stack trace)

**C. Security**
- SQL injection: are `SqlCommand.Parameters` used consistently?
- `ViewState` encryption and MAC validation (WebForms)
- Missing `Authorize` attribute (ASP.NET MVC / Core)

**D. VB.NET-Specific**
- Is `Option Strict On` declared at the top of each file?
- Are `IsNothing()` and `Is Nothing` mixed? (Standardize to one form)
- Is late binding present? (Assignment to `Object` type variables)
- `OracleCommand` with `BindByName = True`: is named binding used consistently?

---

### COBOL Review Checklist

**A. Data Definitions**
- Does the `WORKING-STORAGE` PIC size match the corresponding DB column definition?
- Are the digit counts and data lengths for `COMP-3` (packed decimal) consistent?

**B. SQL**
- Is there a `SQLCODE` check after every `EXEC SQL` block?
- Is `NOT FOUND` (SQLCODE +100) explicitly handled?
- Is `CURSOR` OPEN / FETCH / CLOSE symmetrical?

**C. File Operations**
- Is `FILE STATUS` checked after every `OPEN`?
- Are `CLOSE` statements missing due to control flow interruptions (e.g., `GOTO`)?

---

## Phase 3: Review Report Output

For each file / function, output:

```markdown
## Review Result: [File Name]

### Critical (Fix immediately)
- [Line number] [Finding] — [Remediation approach]

### Warning (Fix recommended)
- [Line number] [Finding]

### Info (Awareness items)
- [Content]

### Summary
[1–3 sentences. Use direct, assertive language. "It appears that…" is prohibited]
```

---

## Forbidden — Absolute Prohibitions (D4)

- **No vague judgments**: Do not say "there may be a risk of…" — say "this is an issue; fix it."
- **No mid-list stops**: Do not stop before all files in the coverage plan have been reviewed.
- **No AI voice**: Do not write verbose preambles like "In order to improve code quality…".
- **No speculative security findings**: Do not flag security issues that have no basis in the actual code.
