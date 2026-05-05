---
name: batch-design-ja
description: Generates Japanese-style batch design specifications (バッチ設計書) for nightly batches, file transfer jobs, and scheduled data processing. Use when designing or documenting Japanese SI batch systems running on JP1 / A-AUTO / Cron / Windows Task Scheduler, producing バッチ設計書 with job net diagrams, restart points, abend recovery procedures, and SLA-driven retry policies. Covers job net hierarchy, file naming rules (世代管理), backup/retention design, abend code categorization, and re-run procedures common in Japanese enterprise SI projects.
---

# Batch Design — Japanese SI Style

> 日本企業の夜間バッチ・ファイル授受ジョブの設計書を生成する。
> ジョブネット図・異常終了時のオペレーション手順・世代管理・リラン手順を含む網羅版。

---

## Phase 0: Scope

Ask the user:

> バッチ設計の対象を教えてください。
>
> **バッチ種別**: 夜間バッチ / ファイル授受 / 月次集計 / 日中オンライン連動 / リカバリ
> **ジョブ管理**: JP1 / A-AUTO / Senju / Cron / Windows Task Scheduler / 手動
> **実行環境**: オンプレ Linux / Windows Server / メインフレーム（JCL）/ クラウド（AWS Batch / Cloud Scheduler）
> **言語**: Shell / PowerShell / COBOL / Java / Python / VB.NET
> **規模**: ジョブ数・処理データ量（件数 / GB）・SLA（何時までに完了）

---

## Phase 1: Question-Driven Setup

**Stage 1 — Basics**
1. バッチ名・実行タイミング（毎日 02:00 など）・SLA は？
2. 入力データ（テーブル / ファイル / API）と出力データは？
3. 想定処理件数・処理時間の目安は？

**Stage 2 — ジョブネット**
- 先行ジョブ・後続ジョブの依存関係
- 並列実行可否（相互排他のリソース有無）
- 条件分岐（曜日 / 月初 / 月末 / 営業日カレンダー）

**Stage 3 — 異常系**
- 異常終了時のオペレータ通知手段（メール / Slack / 電話）
- リラン可否（冪等性が確保されているか）
- 部分リラン単位（ファイル単位 / レコード単位 / 全量再実行）
- バックアウト手順

---

## Phase 2: バッチ設計書テンプレート

```markdown
# バッチ設計書 — [ジョブ名]

## 0. 基本情報
| 項目 | 内容 |
|---|---|
| ジョブ名 | DAILY_SALES_AGG |
| 実行頻度 | 毎日（営業日のみ） |
| 起動時刻 | 02:00 |
| SLA | 04:00 までに完了 |
| ジョブ管理 | JP1 |
| 実行サーバ | batch-srv-01（Linux） |
| 担当者 | [運用担当 / 開発担当] |

## 1. 処理概要
- 入力: 当日売上トランザクション（T_SALES）
- 出力: 日次集計テーブル（T_SALES_DAILY）/ 配信ファイル（sales_YYYYMMDD.csv）
- 処理件数想定: 50万件 / 処理時間目安: 25分

## 2. 前後関係（ジョブネット）
| 順序 | ジョブ名 | 種別 | 待ち合わせ |
|---|---|---|---|
| 1 | PRE_FILE_RECEIVE | ファイル受信 | 01:30 までに到着 |
| 2 | PRE_DATA_VALIDATE | 検証 | 1 完了 |
| 3 | DAILY_SALES_AGG | 本処理 | 2 正常終了 |
| 4 | POST_FILE_SEND | ファイル送信 | 3 正常終了 |
| 5 | POST_NOTIFY | 完了通知 | 4 正常終了 |

## 3. 入出力ファイル命名規則
| ファイル | 命名規則 | 世代管理 | 退避先 |
|---|---|---|---|
| 入力 | sales_input_YYYYMMDD.csv | 7世代 | /backup/input/ |
| 出力 | sales_output_YYYYMMDD.csv | 31世代 | /backup/output/ |
| ログ | DAILY_SALES_AGG_YYYYMMDD.log | 90世代 | /var/log/batch/ |

## 4. 異常終了コード（ABEND）設計
| RC | 区分 | 意味 | オペレータ対応 |
|---|---|---|---|
| 0 | 正常 | 処理完了 | なし |
| 4 | 警告 | 0件処理 | 後続続行・翌朝開発確認 |
| 8 | 業務エラー | バリデーションNG | 入力修正後リラン |
| 12 | システムエラー | DB接続失敗 | インフラ確認後リラン |
| 16 | 致命 | 想定外例外 | 開発者即時連絡 |

## 5. リラン手順
| リラン区分 | 前提条件 | 手順 |
|---|---|---|
| 全量リラン | 出力テーブル/ファイルを退避 | TRUNCATE → 再実行 |
| 増分リラン | チェックポイント保持 | 引数 --resume を指定 |
| 一部リラン | ファイル単位 | 引数 --file=XXX を指定 |

## 6. リソース要件
| 項目 | 値 |
|---|---|
| CPU | 2コア以上 |
| メモリ | 4GB |
| ディスク | 一時領域 10GB |
| DB セッション | 排他取得不要 |

## 7. 監視・通知
| 監視項目 | 閾値 | 通知先 |
|---|---|---|
| 実行時間 | 60分超過 | 運用Slack |
| 異常終了 | RC ≧ 12 | 運用電話 + 開発メール |
| 0件処理 | RC = 4 | 開発メール |

## 8. TODO / 要確認事項
```

---

## 営業日カレンダー / 月末月初の扱い

設計書に必ず含める観点:

- 営業日判定（祝日テーブル / 第N営業日）の参照先
- 月末処理（月末日付の取得方法・閏日対応）
- 年度切替（4月1日 / 1月1日）の特別処理有無

```markdown
## 営業日判定
- 参照: M_BUSINESS_DAY テーブル
- 非営業日の場合: スキップ（前営業日に繰上げ実行はしない）
- 月末判定: M_BUSINESS_DAY.IS_MONTH_END = '1' の日のみ実行
```

---

## ジョブネット記法（テキスト図）

```
┌──────────────────┐
│ 01:30 ファイル到着待ち    │
└──────┬─────────────┘
       │
┌──────▼─────────────┐
│ 02:00 PRE_FILE_RECEIVE   │
└──────┬─────────────┘
       │
┌──────▼─────────────┐
│ 02:05 PRE_DATA_VALIDATE  │ (RC=8 → 異常終了 → オペ通知)
└──────┬─────────────┘
       │
┌──────▼─────────────┐
│ 02:10 DAILY_SALES_AGG    │ (本処理 / 25分)
└──────┬─────────────┘
       │
┌──────▼─────────────┐
│ 02:40 POST_FILE_SEND     │
└──────┬─────────────┘
       │
┌──────▼─────────────┐
│ 02:45 POST_NOTIFY        │ (Slack 完了通知)
└──────────────────┘
```

---

## Forbidden

- **異常終了コード設計の省略禁止** — RC=0 だけでなく警告/業務/システム/致命の階層を必ず定義
- **リラン手順の省略禁止** — 冪等性の有無を明記し、再実行可否と単位を記述
- **世代管理の省略禁止** — ファイル退避世代数を必ず明記（バックアップ起点になる）
- **SLA 未記載禁止** — 完了時刻と超過時のエスカレーション先を必ず書く
- **営業日カレンダーの参照先を曖昧にしない** — テーブル名 / API / ファイルを明示
- **推測断定禁止** / **途中停止禁止** / **AI voice禁止**
