---
name: code-review-typescript
description: Structured code review for TypeScript and React/Next.js codebases with security-first checklists. Use when reviewing TypeScript PRs, auditing a Next.js or NestJS application before deployment, checking TypeScript code quality, or assessing React component architecture. Extends code-review-enterprise-ja. Covers type safety (any/unknown/as), async error handling, React hooks (useEffect deps, key prop), XSS via dangerouslySetInnerHTML, SQL injection in raw queries, and environment variable exposure.
---

# Code Review — TypeScript / React / Next.js

> Full coverage across 5 languages: install **code-review-enterprise-ja**.
> This skill focuses on TypeScript / React / Next.js.

---

## Activation

Ask: レビュー対象のファイル名または機能名を教えてください。Next.js App Router / Pages Router どちらですか？

---

## Coverage Plan

List all files before starting. Do not self-prioritize.

---

## TypeScript Review Checklist

### A. 型安全

- `any` 型の使用箇所 → `unknown` への置き換え候補
- `as` キャスト（型アサーション）の乱用 → 型ガード関数への置き換え
- `!` Non-null assertion → null チェックに置き換え可能か
- `object` / `{}` 型の過度な使用 → 具体的な型定義に変更
- `interface` vs `type` の一貫性（プロジェクト方針に従っているか）

### B. 非同期処理

- `async/await` に対応した `try/catch` があるか
- `Promise.all` / `Promise.allSettled` のエラーハンドリング
- `useEffect` 内の非同期処理: `async function` を内部定義しているか（直接 async は禁止）
- fetch の失敗（4xx/5xx）を `response.ok` で確認しているか

### C. セキュリティ

- `dangerouslySetInnerHTML` の使用 → XSS リスク。sanitize ライブラリの使用確認
- `process.env` のクライアント側露出 → `NEXT_PUBLIC_` prefix のない変数がクライアントバンドルに含まれていないか
- SQL インジェクション: `prisma.$queryRaw` / `pg.query` でのテンプレートリテラル使用 → パラメータバインドに変更
- `eval()` / `Function()` / `new Function()` の使用

### D. React / Next.js 固有

- `useEffect` の依存配列漏れ（ESLint `exhaustive-deps` で検出可能）
- `key` プロップに配列インデックスを使用 → ユニークIDに変更（並び替えがある場合）
- コンポーネントサイズ（200行超は分割検討）
- Server Component / Client Component の境界（`"use client"` の適切な配置）
- `getServerSideProps` での機密データ漏洩（クライアントに渡す `props` に secrets が含まれていないか）

### E. パフォーマンス

- `useMemo` / `useCallback` の過剰使用（逆にパフォーマンス低下）
- 大きな配列の `map` 内でのオブジェクト生成（メモ化候補）
- Next.js Image: `<img>` タグの直接使用（`<Image>` に置き換え推奨）

---

## Review Report Output

```markdown
## レビュー結果: [ファイル名]

### 重大（即修正）
- [行番号] `as unknown as SomeType` — 型安全を迂回している。型ガード関数を実装せよ

### 警告（修正推奨）
- [行番号] `useEffect` 依存配列に `fetchUser` が不足

### 情報
- `interface` と `type` が混在。プロジェクト規約を統一することを推奨

### 総評
[断定口調で1〜3行]
```

---

## Forbidden

- **「〜の可能性があります」禁止** — 「〜である。修正せよ」と断定する
- **途中停止禁止** — 全ファイルレビュー完了まで止まらない
- **AI voice禁止**
