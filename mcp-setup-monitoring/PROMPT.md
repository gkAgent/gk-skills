# MCP 監視ツール セットアップ (Datadog / Sentry)

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/mcp-setup-monitoring` で自動起動します（貼り付け不要）。

---

以下の指示に従って、監視ツールの MCP サーバーをセットアップしてください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: 監視ツールの選択

まず以下を質問してください:

> 使っている監視ツールを教えてください。
>
> **ツール**: Datadog / Sentry
>
> また、主にどんな用途で Claude Code から使いたいですか？（複数可）
>
> 1. エラーログの調査・集計
> 2. メトリクス / パフォーマンス確認
> 3. アラート・インシデント確認
> 4. リリース後の影響確認

---

## Phase 1: セットアップと .mcp.json 生成

### Datadog

**Step 1 — API キーと Application キーを取得する**

1. Datadog → Organization Settings → **API Keys** → New Key
2. Datadog → Organization Settings → **Application Keys** → New Key

**必要な権限スコープ:**

| やりたいこと | 必要なスコープ |
|---|---|
| ログ検索 | `logs_read_data`, `logs_read_index_data` |
| メトリクス取得 | `metrics_read` |
| モニター / アラート確認 | `monitors_read` |
| インシデント確認 | `incident_read` |

**Step 2 — .mcp.json を作成する**

```json
{
  "mcpServers": {
    "datadog": {
      "command": "npx",
      "args": ["-y", "@winor30/mcp-server-datadog"],
      "env": {
        "DATADOG_API_KEY": "your_api_key_here",
        "DATADOG_APP_KEY": "your_application_key_here",
        "DATADOG_SITE": "datadoghq.com"
      }
    }
  }
}
```

**`DATADOG_SITE` の値:**

| リージョン | 値 |
|---|---|
| US1 (デフォルト) | `datadoghq.com` |
| EU1 | `datadoghq.eu` |
| 日本 (JP1) | `ap1.datadoghq.com` |

---

### Sentry

**Step 1 — Auth Token を取得する**

1. Sentry → Settings → Account → **API → Auth Tokens** → Create New Token
2. 必要なスコープ: `event:read`, `project:read`, `releases`, `member:read`, `org:read`

**Step 2 — .mcp.json を作成する**

```json
{
  "mcpServers": {
    "sentry": {
      "command": "npx",
      "args": ["-y", "mcp-server-sentry"],
      "env": {
        "SENTRY_AUTH_TOKEN": "sntrys_XXXXXXXXXXXX",
        "SENTRY_ORGANIZATION": "your-org-slug",
        "SENTRY_PROJECT": "your-project-slug"
      }
    }
  }
}
```

**配置先:** プロジェクトルートの `.mcp.json`。`.gitignore` に必ず追加すること。

---

## Phase 2: 接続テスト

**Datadog:**

```
Datadogで過去1時間のエラーログを10件取得してください
```

**Sentry:**

```
Sentryで未解決のIssueを優先度順に10件一覧にしてください
```

**よくあるエラー:**

| エラー | 原因 | 対処 |
|---|---|---|
| `403 Forbidden` | APIキー / トークンのスコープ不足 | キーを再生成して必要スコープを追加 |
| `Invalid site` (Datadog) | `DATADOG_SITE` が間違い | リージョンに合わせた値を設定 |
| `Organization not found` (Sentry) | org slug が間違い | Sentry URL の `/organizations/YOUR-SLUG/` を確認 |

---

## Phase 3: 使用例

### Datadog — ログ調査

```
本番環境で今日の14:00〜15:00の間に service:api-server で ERROR レベルのログを検索して、
最も頻出するエラーメッセージをランキングにしてください
```

```
昨日のデプロイ後にエラーレートが上がっていないか確認してください。デプロイ前後1時間を比較してください
```

### Sentry — エラー分析

```
過去7日間で最も多く発生しているエラーTop5を教えてください。
それぞれスタックトレースを見て、コードのどこが原因か特定してください
```

```
今週リリースしたバージョン v2.3.1 で新たに発生したエラーだけ抽出して一覧にしてください
```

---

## 絶対禁止事項

- **APIキーをソースコードにハードコードしない**: 必ず `env` フィールドまたは環境変数で管理する
- **.mcp.json を公開リポジトリにコミットしない**: secrets が含まれる。`.gitignore` に追加すること
- **Write 権限スコープを不用意に付与しない**: 監視ツールへの書き込みは意図せず実行されるリスクがある
- **個人情報が含まれるログをClaudeに渡さない**: ユーザーのメールアドレス・IPアドレス等は事前にマスク処理すること
