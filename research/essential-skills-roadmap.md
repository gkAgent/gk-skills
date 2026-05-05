# Essential Skills Roadmap — gkAgent v1.0 GA 向け統合計画

> 翠 (AI/Prompt ENG) 起案 / 2026-05-03 ドラフト / 律監査委員会レビュー前
> 対象: Boris Cherny 30 Tips × Anthropic 公式 10 skill × gk-skills 既存 50 件
> ゴール: 「使うべき / 取り込むべき / 削るべき / 新規実装すべき」の 4 区分整理

---

## 0. エグゼクティブサマリ

| 区分 | 件数 | 備考 |
|---|---:|---|
| **必須 skill (Tier-S)** | **9** | 誰もが入れるべき・gkAgent 標準セット |
| **gkAgent 固有 (Tier-A)** | **18** | 既存 50 件のうち本当に光る業務系 |
| **取り込むべき公式 skill** | **6** | Anthropic 公式 10 のうち事業に直結 |
| **削除候補 (要承認)** | **17** | 重複・粒度過剰・更新断絶 |
| **新規実装すべき (Boris 由来)** | **8** | Plan/Verify/Persist/Automate 補強 |
| **v1.0 GA 目標件数** | **41** | 50 → -17 + 18 (+8 新規 含む) ≒ 41 |

**律監査チェック**: 競合固有名詞・比較広告・誇大表現・一人称複数表現の検出ゼロ。HBA 内部固有名詞も非掲載を確認。

---

## 1. 既存 gk-skills 50 件 カテゴリ別棚卸し

| カテゴリ | 件数 | 中核 | 重複・冗長 |
|---|---:|---|---|
| spec-gen | 16 | enterprise-ja, enterprise(EN), java-oracle, csharp-oracle, vbnet-sqlserver, cobol-oracle | 言語×DB の細分化が 14 本 (削減余地大) |
| code-review | 7 | enterprise-ja, java, csharp, typescript, vbnet, cobol | enterprise (EN版) は -ja に統合可 |
| migration | 5 | vbnet-typescript, csharp-typescript, vbnet-aspnetcore, cobol-typescript, db-migration-oracle-postgresql | + svn-git は migration カテゴリに |
| migration (拡張) | 1 | migration-svn-git | 単独保持 |
| test-gen | 3 | enterprise-ja, csharp-xunit, vbnet-xunit | xunit 個別は enterprise-ja に吸収可 |
| MCP setup | 7 | db-all, db, github, monitoring, cloudflare, communication, audit + enterprise-ja | db-all と db は重複ぎみ |
| 品質/性能 | 3 | security-review-web, perf-review-sql, refactor-enterprise-ja | 維持 |
| API/docs | 3 | api-spec-openapi, onboarding-doc-ja, incident-rca-ja | 維持 |
| 運用 | 3 | todo-resolution-ja, batch-design-ja, log-format-ja | 維持 |
| 横断 (TEXTBOOK_MAPPING.md) | 1 | 60 章教本連携表 | skill ではないが残置 |

実 skill ディレクトリ数 = **49** (TEXTBOOK_MAPPING.md を除く)。

---

## 2. 必須 skill (Tier-S) — 誰もが入れる 9 本

業務システム開発者が最初の 1 時間で入れるべき最小セット。

| # | skill | 由来 | Why |
|---|---|---|---|
| S1 | spec-gen-enterprise-ja | 既存 | 全言語×全DB を 1 本で覆う旗艦 |
| S2 | code-review-enterprise-ja | 既存 | 言語横断レビューの基準点 |
| S3 | test-gen-enterprise-ja | 既存 | xunit/JUnit/Jest 横断 |
| S4 | todo-resolution-ja | 既存 | 仕様 TODO 自動巡回 (社長実証済) |
| S5 | security-review-web | 既存 | OWASP Top10 機械チェック |
| S6 | mcp-audit | 既存 | .mcp.json セキュリティ診断 |
| S7 | **plan-mode-kit** (新規) | Boris #1 | 重い変更の Plan/Implement 分離テンプレ |
| S8 | **self-verify-kit** (新規) | Boris #2 | テスト/CLI/スクショで Claude 自己検証 |
| S9 | **claudemd-curator** (新規) | Boris #4,#11 | ミス→CLAUDE.md 追記の自動巡回 |

---

## 3. gkAgent 固有 skill (Tier-A) — 18 本

業務系 SI で「ここだけ痛みが消える」核。

spec-gen 系 (5):
spec-gen-java-oracle / spec-gen-csharp-oracle / spec-gen-vbnet-sqlserver / spec-gen-vbnet-oracle / spec-gen-cobol-oracle

code-review 系 (5):
code-review-java / code-review-csharp / code-review-typescript / code-review-vbnet / code-review-cobol

migration 系 (5):
migration-vbnet-typescript / migration-csharp-typescript / migration-vbnet-aspnetcore / migration-cobol-typescript / db-migration-oracle-postgresql

業務深掘り (3):
batch-design-ja / log-format-ja / incident-rca-ja

---

## 4. 取り込むべき公式 skill — 6 本

Anthropic 公式 10 のうち事業直結のみ採用。残り 4 (stripe / typefully / composio / cloudflare-app) は様子見。

| # | 公式 skill | 用途 | gkAgent 内での連携 |
|---|---|---|---|
| O1 | skill-creator | skill 雛形生成 | gk-skills の品質基盤・雛形統一 |
| O2 | mcp-builder | MCP サーバー雛形 | gk-test-mcp / gkAgent 内製 MCP |
| O3 | pptx | PowerPoint 出力 | 提案書・社内デモ資料 |
| O4 | docx | Word 出力 | 仕様書清書・顧客提出版 |
| O5 | xlsx | Excel 出力 | gkChouhyou / 帳票・台帳 |
| O6 | pdf | PDF 出力 | gkReport / 公式配布資料 |

採用見送り (要再評価): stripe (収益拡大後), typefully (X 自動投稿は律監査要), composio (依存度高), cloudflare 公式 (mcp-setup-cloudflare が同領域をカバー済)。

---

## 5. 削除候補 — 17 本 (社長承認待ち)

理由は「冗長 = 旗艦に統合可」「英語版は別リポへ分離」「粒度過剰」のいずれか。

英語版分離 (2):
spec-gen-enterprise / code-review-enterprise → 英語圏向け gk-skills-en リポへ切り出し

spec-gen 細分化の整理 (10): 旗艦 spec-gen-enterprise-ja に吸収・section リンク化:
spec-gen-java-postgresql / spec-gen-java-mysql / spec-gen-java-sqlserver /
spec-gen-csharp-postgresql / spec-gen-csharp-sqlserver /
spec-gen-typescript-postgresql / spec-gen-typescript-mysql / spec-gen-typescript-sqlserver /
spec-gen-vbnet-postgresql / spec-gen-cobol-db2

test-gen 個別 (2): test-gen-csharp-xunit / test-gen-vbnet-xunit → enterprise-ja に吸収

MCP 重複 (1): mcp-setup-db (db-all に内包)

その他 (2): mcp-setup-enterprise-ja (個別 setup でカバー可) / migration-svn-git (頻度低・aidev-guide 教本へ)

**注**: 全件 README からは「sunset 予定」と告知するのみ。物理削除は社長承認後。

---

## 6. 新規実装すべき skill — 8 本 (Boris Tips 由来)

| # | skill 案 | Boris Tip | 担当 |
|---|---|---|---|
| N1 | plan-mode-kit | #1 Plan Mode | 海斗+翠 |
| N2 | self-verify-kit | #2 自己検証 | 翠+詩織 |
| N3 | claudemd-curator | #4 / #11 CLAUDE.md 編集 | 楓+詩織 |
| N4 | permissions-bootstrap | #7 allow/ask/deny 雛形 | 颯+律 |
| N5 | hook-installer-ja | #10 / #24 PostToolUse formatter / 律 NG 強制 | 颯+律 |
| N6 | requirement-interview-ja | #15 仕様曖昧時の Claude 逆質問 | 翠+空 |
| N7 | analyze-cli-kit | #14 CLI 経由分析タスク | 蓮+悠希 |
| N8 | fanout-runner | #21 / #22 claude -p fan-out | 颯+悠希 |

各 skill 着手時は skill-creator (公式) で雛形生成 → audit_check.py 通過 → 律監査委員会レビューの順。

---

## 7. v1.0 GA 計画

### 7.1 件数推移
50 (現在) → -17 (削除候補) + 18 (gkAgent 固有再認定) は重複ノーカウント = 33 → +8 (新規) = **41**

### 7.2 マイルストーン

| 期日 | アクション | 担当 |
|---|---|---|
| 5/3 (本日) | 本ロードマップ社長承認 | 翠 → 律 → 櫂 |
| 5/3-5/6 (GW) | Tier1 Boris Tips 5 件 (N1, N2, N3, N4, N5) skill 化着手 | 翠+蓮+颯 |
| 5/7-5/13 | Tier2 Boris Tips 3 件 (N6, N7, N8) + 公式 skill 6 件取り込み | 翠+暁 |
| 5/14-5/20 | 既存 50 件の品質磨き (説明文・再現性・連携) | 蓮+楓 |
| 5/21-5/27 | 削除候補 17 件 sunset 告知 + README 改訂 | 楓+詩織 |
| 5/28-5/31 | v1.0 GA リリース判定 (audit_check.py 全通過) | 律監査委員会 |

### 7.3 v1.0 GA 基準
- 41 skill 全件 audit_check.py = 終了コード 0
- 律監査委員会 4 名合議で「対外発信 OK」サインオフ
- README に「使うべき 9 本 / 業務 18 本 / 公式 6 本 / 新規 8 本」明示
- 公開前トリプルチェック (蒼 → 律 → 櫂 → 社長) 通過

---

## 8. 律監査チェックポイント (本書自体)

- 競合固有名詞: なし
- 比較広告 (「他社より」「××より速い」): なし
- 誇大表現 (最上級・断定強調語): 検出なし (「核」「旗艦」は内部用語のみ)
- 一人称複数表現: 本書は内部資料のため使用していないことを再確認 (対外公開時も置換不要)
- HBA 等 社内固有名詞: なし
- 17 年キャリア / © 表記: なし

---

## 9. 次アクション (社長依頼ベース)

1. 本ロードマップを社長に提示 (LINE 報告は櫂経由)
2. 削除候補 17 件の sunset 告知について 🔴 REDリスト承認要求
3. 新規 8 件の skill 着手は 🟢 自動実行可 (内製のみ・公開なし)
4. 公式 skill 6 件の取り込みは 🟢 自動実行可 (依存追加なし)
5. v1.0 GA リリース告知は 🔴 REDリスト承認要求 (5/28 以降)

---

(End of essential-skills-roadmap.md)
