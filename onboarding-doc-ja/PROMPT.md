# オンボーディングドキュメント生成 — 日本SI向け

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/onboarding-doc-ja` で自動起動します（貼り付け不要）。

---

以下の指示に従って、新規参画者向けのオンボーディングドキュメントを生成してください。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 対象を教えてください。
>
> **システム**: アプリ名・業務領域（販売管理 / 在庫管理 / 人事 等）
> **言語/FW**: Java Spring Boot / VB.NET WinForms / TypeScript Next.js 等
> **DB**: Oracle / PostgreSQL / MySQL / SQL Server
> **読者**: 新規参画のエンジニア / 業務引き継ぎ担当 / 外部ベンダー

---

## システム概要書テンプレート

以下の構造でドキュメントを生成してください:

```markdown
# [システム名] — システム概要書

**最終更新**: YYYY-MM-DD
**対象読者**: 新規参画エンジニア

---

## 1. システム概要

| 項目 | 内容 |
|---|---|
| システム名 | [システム名] |
| 業務領域 | [例: 販売管理 / 在庫管理 / 人事給与] |
| 利用部門 | [例: 営業部・経理部] |
| 稼働環境 | [例: Windows Server 2022 / IIS 10 / Oracle 19c] |
| 言語/FW | [例: VB.NET 4.8 / WinForms] |
| 開発チーム規模 | [例: 5名（SE×2 + PG×3）] |

### システムの目的
[業務上の目的を2〜3文で記述。技術説明ではなく業務説明]

---

## 2. アーキテクチャ概要

```
[クライアント層]
WinForms アプリ (C# / VB.NET)
    ↓ ODP.NET / ADO.NET
[データ層]
Oracle 19c（本番）/ Oracle XE（開発）
```

### 主要コンポーネント
| コンポーネント | 役割 | 技術スタック |
|---|---|---|
| UI 層 | 画面表示・入力受付 | WinForms / WebForms |
| サービス層 | ビジネスロジック | C# クラスライブラリ |
| データ層 | DB アクセス | ODP.NET / EF Core |

---

## 3. 開発環境構築

### 必要なツール
| ツール | バージョン | 入手先 |
|---|---|---|
| Visual Studio | 2022 Community | [URL] |
| Oracle Client | 19c | 社内共有ドライブ |
| Git | 2.x | git-scm.com |

### セットアップ手順
```bash
# 1. リポジトリ取得
git clone https://[リポジトリURL]

# 2. 接続文字列設定
cp config/appsettings.Development.json.example config/appsettings.Development.json
# appsettings.Development.json の ConnectionStrings.Default を編集

# 3. ビルド・実行
dotnet run --project src/MyApp.csproj
```

---

## 4. ディレクトリ構成

```
MyApp/
├── src/
│   ├── MyApp.UI/           # WinForms プロジェクト（画面）
│   ├── MyApp.Service/      # ビジネスロジック
│   ├── MyApp.Repository/   # DB アクセス
│   └── MyApp.Domain/       # エンティティ・値オブジェクト
├── tests/
└── config/
    └── appsettings.json    # 接続文字列・設定（本番値は環境変数）
```

---

## 5. 主要テーブル一覧

| テーブル名 | 日本語名 | 主キー | 行数（概算） | 備考 |
|---|---|---|---|---|
| EMPLOYEE | 社員マスタ | EMP_ID | 5,000件 | 論理削除（IS_DELETED フラグ） |
| SALES_ORDER | 受注ヘッダ | ORDER_ID | 200万件 | パーティション（年次） |

---

## 6. よくある開発作業

### 画面追加の手順
1. UI プロジェクトに新しい Form クラスを追加
2. Service プロジェクトに対応するサービスメソッドを追加
3. Repository プロジェクトに SQL クエリを追加
4. 単体テストを追加

---

## 7. 注意事項・ハマりやすいポイント

| 項目 | 内容 |
|---|---|
| Oracle DATE 型 | 時刻情報を含む。日付比較は TRUNC() を使うこと |
| ODP.NET BindByName | `cmd.BindByName = True` を忘れると誤ったパラメータがバインドされる |
| 論理削除 | EMPLOYEE テーブルは物理削除しない。IS_DELETED = 1 で無効化 |

---

## 8. 連絡先・リソース

| リソース | 場所 |
|---|---|
| 詳細設計書 | SharePoint: [URL] |
| 課題管理 | Redmine: [URL] |
| 担当者 | [PM 名前] (PJ 全体) / [TL 名前] (技術) |
```

---

## 生成のポイント

既存コードから読み取る情報:
- `application.yml` / `web.config` → 接続先・環境設定
- `pom.xml` / `*.csproj` → 技術スタック・バージョン
- ディレクトリ構成 → アーキテクチャ
- SQL / Mapper → 主要テーブル一覧

---

## 絶対禁止事項

- **接続文字列の本番値を文書に記載禁止** — 「環境変数参照」または「Confluence 参照」と記述
- **業務説明なし（技術説明のみ）禁止** — 新規参画者が「何のためのシステムか」を理解できる説明を必ず含める
- **途中停止禁止** / **AI voice禁止**
