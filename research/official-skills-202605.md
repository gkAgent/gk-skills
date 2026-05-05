# 公式 Skill 10 件 精査レポート

- 調査者: 暁 (MCP Researcher)
- 日付: 2026-05-02
- 想定読者: gkAgent 経営層 (社長 / 櫂 / 律 / 蒼)
- 監査ライン: 律監査委員会 (出版前チェック想定)

> 本レポートは「Anthropic および主要事業者が公開する Agent Skill 10 件」の精査結果を観測ベースで記述したものです。優劣比較や競合直接比較は行いません。

---

## 1. 調査結果一覧表

| # | Skill | 提供元 | 公開リポジトリ | 主要機能 (5 行以内) | ライセンス | 取り込み判定 |
|---|---|---|---|---|---|---|
| 1 | skill-creator | anthropics | github.com/anthropics/skills (skills/skill-creator) | Q&A 形式で SKILL.md・スクリプト・参照資料を生成。スキル開発の本家ガイド。 | リポジトリ準拠 | ⭐⭐⭐ |
| 2a | pptx | anthropics | github.com/anthropics/skills (skills/pptx) | テンプレート/スクラッチ両対応。マスターレイアウト・スピーカーノート・コメント対応。 | リポジトリ準拠 | ⭐⭐⭐ |
| 2b | docx | anthropics | github.com/anthropics/skills (skills/docx) | .docx の ZIP/XML 直接編集。目次・ヘッダーフッター・画像・変更履歴・置換。 | リポジトリ準拠 | ⭐⭐⭐ |
| 2c | xlsx | anthropics | github.com/anthropics/skills (skills/xlsx) | 数式エラー回避を志向。チャート・条件付き書式・ピボット対応。 | リポジトリ準拠 | ⭐⭐⭐ |
| 2d | pdf | anthropics | github.com/anthropics/skills (skills/pdf) | テキスト抽出・結合分割・透かし・フォーム入力・暗号化・OCR。 | リポジトリ準拠 | ⭐⭐⭐ |
| 3 | mcp-builder | anthropics | github.com/anthropics/skills (skills/mcp-builder) | Python (FastMCP)/Node (TS SDK) MCP サーバーを 4 フェーズで構築するガイド。評価手順含む。 | リポジトリ準拠 | ⭐⭐⭐ |
| 4 | stripe-best-practices | stripe | github.com/stripe/ai (skills/stripe-best-practices) ・ docs.stripe.com/.well-known/skills | Checkout/PaymentIntents/Connect/Subscriptions/Webhook/鍵管理の選択指針を提供。 | リポジトリ準拠 | ⭐ |
| 5 | next-best-practices (実体: vercel-react-best-practices ほか) | vercel-labs | github.com/vercel-labs/agent-skills | React/Next.js のパフォーマンス・アーキ・プラットフォーム最適化ルール集。 | リポジトリ準拠 | ⭐⭐ |
| 6 | workers-best-practices | cloudflare | github.com/cloudflare/skills | Workers/KV/R2/D1/Wrangler のベストプラクティスとアンチパターン回避指針。 | リポジトリ準拠 | ⭐⭐⭐ |
| 7 | gemini-api-dev | google-gemini | github.com/google-gemini/gemini-skills (skills/gemini-api-dev) | Gemini API/SDK の Python・TS・Java・Go 利用ガイド。モデル選定指針あり。 | リポジトリ準拠 | ⭐ |
| 8 | typefully | typefully | github.com/typefully/agent-skills | X / LinkedIn / Threads / Bluesky / Mastodon の下書き・予約投稿を Typefully API 経由で実施。 | MIT (確認済) | ⭐⭐ |
| 9 | remotion (skill 群) | remotion-dev | github.com/remotion-dev/skills | Remotion (React 動画フレームワーク) の 28+ ルールを動的読込みし、プログラマティック動画生成を支援。 | リポジトリ準拠 | ⭐ |
| 10 | composio | composiohq | github.com/composiohq/composio (および ComposioHQ/composio-plugin-cc) | 1000+ SaaS への managed auth/ツールルーティング/サンドボックス実行を提供。 | リポジトリ準拠 | ⭐⭐ |

> 注: 「next-best-practices」は社長が朝拾った文脈での通称で、公式リポジトリでの該当 Skill 名は `vercel-react-best-practices` 等の個別 Skill 群として提供される運用が観測された。

---

## 2. 取り込み優先度サマリ

### ⭐⭐⭐ 即取り込み (5/3 〜 5/9 想定): 6 件
- skill-creator
- pptx / docx / xlsx / pdf (4 件セット)
- mcp-builder
- workers-best-practices

### ⭐⭐ 中期 (5 月後半 〜 6 月想定): 3 件
- vercel-labs/agent-skills (Next.js / React)
- typefully (社外発信を陽菜 CMO 主導で運用検討)
- composio (managed auth で個別 MCP 実装の補完候補)

### ⭐ 将来検討: 2 件
- stripe-best-practices (現時点で Stripe 決済導入予定なし)
- gemini-api-dev (主モデルは Claude のため副次)
- remotion (動画は Phase 2 以降の検討枠)

---

## 3. gkAgent 製品別 活用シーン候補

| 製品 / プロジェクト | 活用候補 Skill | 想定シーン |
|---|---|---|
| aidev-guide / 教本 / Note 販売 | pptx, docx, pdf | 章立て資料・配布 PDF・解説スライドの生成補助 |
| aidev-guide 集計 | xlsx | 売上集計・KPI ダッシュボード作成補助 |
| gk-skills 自体 | skill-creator | 既存 50 件のリファクタ指針および新規 Skill の雛形生成 |
| gk-skills (mcp-setup-* 系) | mcp-builder | 自作 MCP サーバー (LINE 双方向化等) の品質基準導入 |
| infra/gkagent-hp | vercel-labs (Next/React 系) | 将来 HP リニューアル時の Web 品質ガイド |
| LINE Bot 双方向化 (設計 2026-04-28) | workers-best-practices, composio | Cloudflare Workers + KV/R2 構成のガイド + 外部 SaaS 連携の補完 |
| SNS 運用 (Threads / X / note) | typefully | 陽菜 CMO の予約投稿運用基盤候補 (LINE 承認フロー前提) |
| jichitai-intel / gk-intel 系 | composio | 案件ヒアリング段階で外部 SaaS 連携要件が出た場合の保険 |

---

## 4. 既存 50 件との関係 (重複 / 補完)

### 重複が観測された領域: 1 件
- `gk-skills/mcp-setup-cloudflare` と `cloudflare/skills (workers-best-practices)`
  - 重複度: 中。前者は MCP セットアップ手順、後者はコード品質ガイドで、補完関係に近い。並存運用が無難。

### 補完が期待される領域 (重複なし): 5 件
- `mcp-builder` ↔ 既存 `mcp-audit` / `mcp-setup-*` (自作 MCP の品質基準を上流から強化)
- `pptx` / `docx` / `pdf` ↔ 既存 `onboarding-doc-ja` / `incident-rca-ja` (出力フォーマット拡張)
- `xlsx` ↔ 既存帳票・集計系 (gkReport 構想と連結可能)
- `vercel-labs` ↔ Frontend 凛のフロー (新規 LP/HP 製造時の品質ベース)
- `composio` ↔ 既存 `mcp-setup-communication` (managed auth の置換ではなく補完評価)

### 重複なし (新規領域): 4 件
- `skill-creator` (gk-skills のメタ運用)
- `stripe-best-practices` (決済導入時に新規)
- `gemini-api-dev` (副モデル運用時に新規)
- `typefully` / `remotion` (社外発信・動画は新規領域)

---

## 5. 「本当に必要な Skill」の絞り込み案

### 5/3 〜 5/9 着手提案 (即取り込み枠)
1. **skill-creator** — gk-skills 新規追加・リファクタの土台
2. **mcp-builder** — LINE Bot 双方向化など自作 MCP の品質基準
3. **pdf / docx / xlsx / pptx (4 件セット)** — Note / 配布 / 集計の出力品質底上げ
4. **workers-best-practices** — Cloudflare Workers 構成の前提知識統一

着手手順案 (蓮 + 海斗 想定):
- products/gk-skills/external/ ディレクトリを新設し、公式 Skill を `git submodule` または個別 clone で取り込み
- gk-skills/README.md に「公式 Skill 連携」節を追加 (出版前トリプルチェック対象)
- audit_check.py 機械チェック (該当する場合) を通過させる

### 5 月後半 〜 6 月着手提案 (中期枠)
- **vercel-labs (Next/React)** — gkagent-hp / 凛のフロー導入時に評価
- **typefully** — 陽菜 CMO 主導で運用ポリシー策定後に試験導入。LINE 承認フロー必須
- **composio** — managed auth が必要な要件が顕在化した時点で導入評価

### 見送り推奨 (現時点)
- **stripe-best-practices** / **gemini-api-dev** / **remotion** — 事業ロードマップとの一致が弱いため、需要発生時に再評価

---

## 6. 律監査ライン チェック

- [x] 競合直接比較なし (OpenAI Functions / LangChain 等の優劣記述なし)
- [x] 一人称複数表現の使用なし
- [x] HBA / 社内固有名詞の混入なし
- [x] 誇大表現なし (「最強」「世界一」等不使用)
- [x] 観測ベースの基調 (「〜を提供」「〜を予定」「〜が観測された」)
- [x] 出典・URL を明示
- [ ] audit_check.py 機械チェック (gk-skills/audit_check.py が現時点で見当たらないため、該当ツール整備後に再実行する想定)

---

## 7. 出典 (主要)

- anthropics/skills: https://github.com/anthropics/skills
- skills/skill-creator/SKILL.md: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md
- skills/mcp-builder/SKILL.md: https://github.com/anthropics/skills/blob/main/skills/mcp-builder/SKILL.md
- stripe/ai (Stripe Skill): https://docs.stripe.com/.well-known/skills/index.json
- vercel-labs/agent-skills: https://github.com/vercel-labs/agent-skills
- cloudflare/skills: https://github.com/cloudflare/skills
- google-gemini/gemini-skills: https://github.com/google-gemini/gemini-skills
- typefully/agent-skills: https://github.com/typefully/agent-skills
- remotion-dev/skills: https://github.com/remotion-dev/skills
- composiohq/composio: https://github.com/composiohq/composio

---

以上。次アクションは社長 / 櫂 の判断待ち。
