# 教本 × gk-skills マッピング表

> **用途:** 「Claude Code 完全実践ガイド（60章教本）」を読んでいる人が、
> その章に対応する gk-skills を見つけるための逆引きガイド。
>
> 教本の詳細は `aidev-guide/docs/claude_code_textbook_design.md` を参照。

---

## Part 1: 基礎 (Ch01-10) — スキルより先に基礎を学ぶフェーズ

Part 1 は「Claude Code とは何か」「インストール」「CLAUDE.md 設計」「Permission」「プロンプト基本」「Git 連携」「コンテキスト管理」を扱う。
この段階では gk-skills よりも教本本文の理解が優先。スキルは Part 2 以降で活きる。

| Chapter | テーマ | 推奨スキル | 使いどころ |
|---|---|---|---|
| Ch01 | Claude Code とは何か | スキル不要 — 教本の解説を参照 | — |
| Ch02 | インストールと初期設定 | スキル不要 — 教本の解説を参照 | — |
| Ch03 | 最初の対話 | スキル不要 — 教本の解説を参照 | — |
| Ch04 | CLAUDE.md の書き方 | スキル不要 — 教本の解説を参照 | — |
| Ch05 | Permission と安全モデル | スキル不要 — 教本の解説を参照 | — |
| Ch06 | プロンプト設計の基本 | スキル不要 — 教本の解説を参照 | — |
| Ch07 | Git 連携 | スキル不要 — 教本の解説を参照 | — |
| Ch08 | コンテキスト管理 | スキル不要 — 教本の解説を参照 | — |
| Ch09 | 実践ワークショップ: 小さなWebアプリ | スキル不要 — 教本の解説を参照 | — |
| Ch10 | Part 1 総復習・トラブルシューティング | スキル不要 — 教本の解説を参照 | — |

---

## Part 2: 中級 (Ch11-20) — 自分の武器にする

| Chapter | テーマ | 推奨スキル | 使いどころ |
|---|---|---|---|
| Ch11 | settings.json 完全攻略 | スキル不要 — 教本の解説を参照 | — |
| Ch12 | スラッシュコマンド大全 | `spec-gen-enterprise-ja` / `code-review-enterprise-ja` | スラッシュコマンドとしての呼び出し実例として活用。「カスタムコマンドとは何か」を体感するのに最適 |
| Ch13 | 効率的なコードベース探索 | `spec-gen-enterprise-ja` / `onboarding-doc-ja` | 大規模コードベースを探索し仕様書を生成する実戦例として活用 |
| Ch14 | テスト駆動AI開発 | `test-gen-enterprise-ja` | TDD サイクルでテスト仕様書とテストコードを自動生成する実戦例として活用 |
| Ch15 | リファクタリング実践 | `refactor-enterprise-ja` | Fat イベントハンドラ→サービス層への整理など、リファクタリングの具体的な指示例として活用 |
| Ch16 | デバッグの極意 | `code-review-enterprise-ja` | バグの原因追跡・コードレビューを通じた静的解析として活用 |
| Ch17 | マルチ言語対応 | スキル不要 — 教本の解説を参照 | — |
| Ch18 | レガシーコード対応 | `refactor-enterprise-ja` / `spec-gen-vbnet-oracle` / `spec-gen-cobol-db2` | ドキュメントのない VB.NET / COBOL を読み解いて仕様書化する実例として活用 |
| Ch19 | ドキュメント自動生成 | `onboarding-doc-ja` / `api-spec-openapi` / `incident-rca-ja` | コードから各種ドキュメントを生成する実例として活用 |
| Ch20 | Part 2 総復習 | スキル不要 — 教本の解説を参照 | — |

---

## Part 3: MCP (Ch21-30) — AIの手足を増やす

MCP 関連スキルは現在構想段階（`予定` 表記）のものを含む。
既存スキルの中では MCP 接続の有無より「エージェント型プロンプト設計」の実例として活用する。

| Chapter | テーマ | 推奨スキル | 使いどころ |
|---|---|---|---|
| Ch21 | MCP とは何か | スキル不要 — 教本の解説を参照 | — |
| Ch22 | MCP サーバーの導入 | `mcp-setup-db`（予定）/ `mcp-setup-github`（予定） | DB / GitHub への MCP 接続セットアップの実例として活用 |
| Ch23 | MCP サーバー自作入門 | スキル不要 — 教本の解説を参照 | — |
| Ch24 | MCP セキュリティ完全ガイド | `mcp-setup-monitoring`（予定） | 監視・監査ログ設計の MCP 構成例として活用 |
| Ch25 | 社内稟議を通す MCP | スキル不要 — 教本の解説を参照 | — |
| Ch26 | レガシーシステムに MCP を入れる | `mcp-setup-db`（予定） | Oracle / レガシー DB への安全な MCP 接続例として活用 |
| Ch27 | マルチ MCP 構成 | `mcp-setup-cloudflare`（予定）/ `mcp-setup-communication`（予定） | 複数 MCP を組み合わせた構成例として活用 |
| Ch28 | MCP サーバー設計パターン集 | スキル不要 — 教本の解説を参照 | — |
| Ch29 | MCP トラブルシューティング | `mcp-audit`（予定） | MCP 接続の問題診断・監査ログ確認の実例として活用 |
| Ch30 | Part 3 総復習 | スキル不要 — 教本の解説を参照 | — |

---

## Part 4: Hooks・Skills・自動化 (Ch31-40) — AIの行動にルールと知恵を埋め込む

**Ch33 は gk-skills の全33スキルが実例として機能する。**
「こういう SKILL.md を書けば、こういう動作になる」というリファレンスとして gk-skills リポジトリ全体を参照。

| Chapter | テーマ | 推奨スキル | 使いどころ |
|---|---|---|---|
| Ch31 | Hooks 入門 | スキル不要 — 教本の解説を参照 | — |
| Ch32 | Hooks 実践パターン集 | スキル不要 — 教本の解説を参照 | — |
| Ch33 | Skills 設計 | **gk-skills 全33スキルが実例** | SKILL.md の設計パターン・トリガー条件・実行手順の参考として gk-skills リポジトリ全体を参照 |
| Ch34 | Hooks + Skills 連携 | `code-review-enterprise-ja` / `security-review-web` | PostToolUse Hook → Skill 自動発火の実例として活用 |
| Ch35 | CronCreate と RemoteTrigger | スキル不要 — 教本の解説を参照 | — |
| Ch36 | GitHub Actions 連携 | スキル不要 — 教本の解説を参照 | — |
| Ch37 | 通知と外部連携 | `security-review-web` | OWASP チェック結果を Slack/LINE に通知するパイプラインの実例として活用 |
| Ch38 | コスト管理と最適化 | スキル不要 — 教本の解説を参照 | — |
| Ch39 | 実践ワークショップ: 自動化パイプライン構築 | `spec-gen-enterprise-ja` / `code-review-enterprise-ja` / `test-gen-enterprise-ja` / `todo-resolution-ja` | spec → review → test → todo 解消の4段階パイプライン構築例として活用 |
| Ch40 | Part 4 総復習 | スキル不要 — 教本の解説を参照 | — |

---

## Part 5: マルチAgent (Ch41-50) — AIチームを率いる

Part 5 は組織設計・ペルソナ設計・Agent 間通信が中心。gk-skills の個別スキルに直接対応する章は少ない。
ただし、gk-skills の各スキルが「専門化された Agent の知識注入例」として設計参考になる。

| Chapter | テーマ | 推奨スキル | 使いどころ |
|---|---|---|---|
| Ch41 | マルチAgent の設計思想 | スキル不要 — 教本の解説を参照 | — |
| Ch42 | Subagents 基礎 | スキル不要 — 教本の解説を参照 | — |
| Ch43 | Agent Teams — AI チーム間通信の設計 | スキル不要 — 教本の解説を参照 | — |
| Ch44 | 組織設計パターン | スキル不要 — 教本の解説を参照 | — |
| Ch45 | Agent ペルソナ設計 | スキル不要 — 教本の解説を参照 | — |
| Ch46 | マルチAgent の MCP 設計 | スキル不要 — 教本の解説を参照 | — |
| Ch47 | エラーハンドリングとリカバリ | `incident-rca-ja` | Agent 障害時の RCA・報告書作成フローとして活用 |
| Ch48 | 実践ワークショップ: 4人体制のAI開発チーム | `spec-gen-enterprise-ja` / `code-review-enterprise-ja` / `test-gen-enterprise-ja` | PM/Developer/Reviewer/Tester の各 Agent に割り当てるスキルの実例として活用 |
| Ch49 | マルチAgent のモニタリングと運用 | `incident-rca-ja` / `perf-review-sql` | 運用中の障害対応・パフォーマンス分析の自動化例として活用 |
| Ch50 | Part 5 総復習 | スキル不要 — 教本の解説を参照 | — |

---

## Part 6: 応用・実戦 (Ch51-60) — 現場で勝つ

Part 6 は gk-skills との対応が最も密。日本 SI 現場での実戦を想定したスキル群がそのまま活きる。

| Chapter | テーマ | 推奨スキル | 使いどころ |
|---|---|---|---|
| Ch51 | 大規模プロジェクトでの Claude Code | `onboarding-doc-ja` / `spec-gen-enterprise-ja` | 10人チームへのナレッジ共有・オンボーディング文書生成として活用 |
| Ch52 | Enterprise 導入ガイド | `spec-gen-enterprise-ja` / `test-gen-enterprise-ja` / `api-spec-openapi` | パイロットチームでの効果測定に spec / test / API ドキュメント自動生成を活用 |
| Ch53 | セキュリティ深掘り | `security-review-web` / `code-review-enterprise-ja` | OWASP Top 10 診断・秘密鍵検出・AI 生成コードの脆弱性レビューとして活用 |
| Ch54 | パフォーマンスチューニング | `perf-review-sql` / `incident-rca-ja` | スロークエリ・N+1 の自動検出と障害報告書作成によるパフォーマンス改善サイクルとして活用 |
| Ch55 | マイグレーション実戦 | `migration-vbnet-typescript` / `migration-csharp-typescript` / `migration-cobol-typescript` / `migration-vbnet-aspnetcore` / `db-migration-oracle-postgresql` | VB.NET・C#・COBOL・Oracle のモダナイゼーション全5パスの実例として活用 |
| Ch56 | API 開発と Claude Code | `api-spec-openapi` / `perf-review-sql` | 既存コードから OpenAPI 3.0 YAML を生成し、DB パフォーマンスも同時に改善するセットとして活用 |
| Ch57 | フロントエンド開発と Claude Code | `spec-gen-typescript-postgresql` / `spec-gen-typescript-mysql` / `spec-gen-typescript-sqlserver` / `code-review-typescript` | TypeScript フロントエンドの仕様書生成・レビューの実例として活用 |
| Ch58 | AI開発のキャリア戦略 | スキル不要 — 教本の解説を参照 | — |
| Ch59 | 実践ワークショップ: 完全プロダクト開発 | `spec-gen-enterprise-ja` / `code-review-enterprise-ja` / `test-gen-enterprise-ja` / `api-spec-openapi` / `security-review-web` / `todo-resolution-ja` | End-to-End プロダクト開発の全工程でスキルを組み合わせて活用 |
| Ch60 | Claude Code の未来 | スキル不要 — 教本の解説を参照 | — |

---

## 逆引き: スキルから章を探す

| スキル | 対応章 |
|---|---|
| `spec-gen-enterprise-ja` | Ch12, Ch13, Ch39, Ch48, Ch52, Ch59 |
| `spec-gen-java-oracle` | Ch13, Ch18, Ch55 |
| `spec-gen-java-postgresql` | Ch13, Ch55 |
| `spec-gen-java-mysql` | Ch13, Ch55 |
| `spec-gen-java-sqlserver` | Ch13, Ch55 |
| `spec-gen-typescript-postgresql` | Ch57, Ch59 |
| `spec-gen-typescript-mysql` | Ch57, Ch59 |
| `spec-gen-typescript-sqlserver` | Ch57, Ch59 |
| `spec-gen-csharp-sqlserver` | Ch13, Ch18, Ch51, Ch55 |
| `spec-gen-csharp-oracle` | Ch13, Ch18, Ch55 |
| `spec-gen-csharp-postgresql` | Ch13, Ch18, Ch55 |
| `spec-gen-vbnet-sqlserver` | Ch13, Ch18, Ch55 |
| `spec-gen-vbnet-oracle` | Ch13, Ch18, Ch55 |
| `spec-gen-cobol-db2` | Ch13, Ch18, Ch55 |
| `code-review-enterprise-ja` | Ch12, Ch16, Ch34, Ch39, Ch48, Ch53, Ch59 |
| `code-review-java` | Ch16, Ch53 |
| `code-review-csharp` | Ch16, Ch53 |
| `code-review-typescript` | Ch16, Ch53, Ch57 |
| `code-review-vbnet` | Ch16, Ch18, Ch53, Ch55 |
| `code-review-cobol` | Ch16, Ch18, Ch53, Ch55 |
| `migration-vbnet-typescript` | Ch55, Ch59 |
| `migration-csharp-typescript` | Ch55, Ch59 |
| `migration-vbnet-aspnetcore` | Ch55, Ch59 |
| `migration-cobol-typescript` | Ch55, Ch59 |
| `db-migration-oracle-postgresql` | Ch55, Ch56, Ch59 |
| `security-review-web` | Ch37, Ch53, Ch59 |
| `perf-review-sql` | Ch49, Ch54, Ch56 |
| `refactor-enterprise-ja` | Ch15, Ch18 |
| `test-gen-enterprise-ja` | Ch14, Ch39, Ch48, Ch52, Ch59 |
| `api-spec-openapi` | Ch19, Ch52, Ch56, Ch59 |
| `onboarding-doc-ja` | Ch13, Ch19, Ch51 |
| `incident-rca-ja` | Ch19, Ch47, Ch49, Ch54 |
| `todo-resolution-ja` | Ch39, Ch59 |

---

## スキルセット推奨

章を読み進める目安として、どのスキルを先にインストールすべきかをまとめた。

### Part 2 到達時点（最初の1本）
```bash
npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja
```

### Part 2 完了時点（SI 標準セット）
```bash
npx skills@latest add gkAgent/gk-skills/spec-gen-enterprise-ja
npx skills@latest add gkAgent/gk-skills/code-review-enterprise-ja
npx skills@latest add gkAgent/gk-skills/test-gen-enterprise-ja
npx skills@latest add gkAgent/gk-skills/todo-resolution-ja
```

### Part 6 Ch53-54 到達時点（品質・セキュリティセット）
```bash
npx skills@latest add gkAgent/gk-skills/security-review-web
npx skills@latest add gkAgent/gk-skills/perf-review-sql
npx skills@latest add gkAgent/gk-skills/refactor-enterprise-ja
npx skills@latest add gkAgent/gk-skills/incident-rca-ja
```

### Part 6 Ch55 到達時点（移行プロジェクトセット）
```bash
# VB.NET 案件の場合
npx skills@latest add gkAgent/gk-skills/migration-vbnet-typescript
npx skills@latest add gkAgent/gk-skills/code-review-vbnet
npx skills@latest add gkAgent/gk-skills/refactor-enterprise-ja

# Oracle → PostgreSQL 案件の場合
npx skills@latest add gkAgent/gk-skills/db-migration-oracle-postgresql
npx skills@latest add gkAgent/gk-skills/spec-gen-java-oracle
npx skills@latest add gkAgent/gk-skills/spec-gen-java-postgresql
```
