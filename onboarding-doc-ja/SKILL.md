---
name: onboarding-doc-ja
description: Generates onboarding documentation (システム概要書 / 開発環境構築手順) for new team members joining Japanese SI projects. Use when creating system overview documents for new developers, generating development environment setup guides from existing configuration files, documenting key business logic and database structure for knowledge transfer, or producing handover documentation when a project changes teams. Covers Java/TypeScript/C#/VB.NET systems with Oracle/PostgreSQL/MySQL/SQL Server.
---

# Onboarding Documentation — Japanese SI

> 新規参画者が3日で戦力になるためのドキュメントを生成する。
> 属人化解消・チーム交代・プロジェクト引き継ぎに。

---

## Phase 0: Scope

Ask the user:

> 対象を教えてください。
>
> **システム**: アプリ名・業務領域（販売管理 / 在庫管理 / 人事 等）
> **言語/FW**: Java Spring Boot / VB.NET WinForms / TypeScript Next.js 等
> **DB**: Oracle / PostgreSQL / MySQL / SQL Server
> **読者**: 新規参画のエンジニア / 業務引き継ぎ担当 / 外部ベンダー

---

## システム概要書テンプレート

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

# 3. DB マイグレーション（EF Core 使用の場合）
dotnet ef database update

# 4. ビルド・実行
dotnet run --project src/MyApp.csproj
```

### 開発環境の接続情報
| 項目 | 値 |
|---|---|
| DB ホスト | dev-oracle.internal:1521 |
| サービス名 | DEVDB |
| ユーザー名 | APP_DEV（各自パスワードは Confluence 参照） |
| Git リモート | https://github.com/[org]/[repo] |

---

## 4. ディレクトリ構成

```
MyApp/
├── src/
│   ├── MyApp.UI/           # WinForms プロジェクト（画面）
│   ├── MyApp.Service/      # ビジネスロジック
│   ├── MyApp.Repository/   # DB アクセス（MyBatis Mapper / EF Core）
│   └── MyApp.Domain/       # エンティティ・値オブジェクト
├── tests/
│   └── MyApp.Tests/        # 単体テスト（xUnit / JUnit）
├── docs/
│   ├── 詳細設計書/          # 画面・テーブル設計書
│   └── ER図/
└── config/
    └── appsettings.json    # 接続文字列・設定（本番値は環境変数）
```

---

## 5. 主要テーブル一覧

| テーブル名 | 日本語名 | 主キー | 行数（概算） | 備考 |
|---|---|---|---|---|
| EMPLOYEE | 社員マスタ | EMP_ID | 5,000件 | 論理削除（IS_DELETED フラグ） |
| DEPARTMENT | 部署マスタ | DEPT_ID | 50件 | |
| SALES_ORDER | 受注ヘッダ | ORDER_ID | 200万件 | パーティション（年次） |
| SALES_DETAIL | 受注明細 | ORDER_ID, LINE_NO | 800万件 | |

---

## 6. よくある開発作業

### 画面追加の手順
1. `MyApp.UI` に新しい Form クラスを追加
2. `MyApp.Service` に対応するサービスメソッドを追加
3. `MyApp.Repository` に SQL（Mapper or EF Core クエリ）を追加
4. 単体テストを `MyApp.Tests` に追加

### デプロイ手順
```bash
# ビルド
dotnet publish -c Release -o ./publish

# IIS 配置（開発サーバー）
xcopy /Y ./publish/* \\dev-server\inetpub\MyApp\
```

---

## 7. 注意事項・ハマりやすいポイント

| 項目 | 内容 |
|---|---|
| Oracle DATE 型 | 時刻情報を含む。日付比較は TRUNC() を使うこと |
| ODP.NET BindByName | `cmd.BindByName = True` を忘れると誤ったパラメータがバインドされる |
| 論理削除 | EMPLOYEE テーブルは物理削除しない。IS_DELETED = 1 で無効化 |
| 接続プール | 開発環境のプールサイズは 5。本番は 20（HikariCP 設定参照） |

---

## 8. 連絡先・リソース

| リソース | 場所 |
|---|---|
| 詳細設計書 | SharePoint: [URL] |
| 課題管理 | Redmine: [URL] |
| CI/CD | Jenkins: [URL] |
| テスト環境 | http://test-server.internal/MyApp |
| 担当者 | [PM 名前] (PJ 全体) / [TL 名前] (技術) |
```

---

## 生成のポイント

既存コードから読み取る情報:
- `application.yml` / `web.config` → 接続先・環境設定
- `pom.xml` / `*.csproj` → 技術スタック・バージョン
- ディレクトリ構成 → アーキテクチャ
- SQL / Mapper → 主要テーブル一覧
- README / 既存コメント → 業務説明

---

## Forbidden

- **接続文字列の本番値を文書に記載禁止** — 「環境変数参照」または「Confluence 参照」と記述
- **業務説明なし（技術説明のみ）禁止** — 新規参画者が「何のためのシステムか」を理解できる説明を必ず含める
- **途中停止禁止** / **AI voice禁止**
