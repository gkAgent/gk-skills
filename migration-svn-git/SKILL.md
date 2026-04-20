---
name: migration-svn-git
description: Migrates SVN repositories to Git with full history and branch preservation. Handles standard (trunk/branches/tags) and non-standard SVN layouts. Generates authors.txt mapping, git-svn clone commands, branch conversion scripts, and team onboarding guide. Use when a Japanese enterprise needs to migrate from SVN to GitHub/GitLab/Gitea, especially for Windows environments with TortoiseSVN users. Covers single and batch (multiple repos) migration.
---

# Migration Kit — SVN to Git

## Activation

When this skill is invoked, identify the migration scope and follow the phase structure below.
Do not skip phases. Do not generate commands before Phase 0 is complete.

---

## Phase 0: SVN 環境確認

まず以下を質問してください:

> 移行環境を教えてください。
>
> **SVN サーバ**: URL（例: http://svn-server/repos/PROJECT）/ VisualSVN Server / その他
> **レイアウト**: 標準（trunk/branches/tags）/ 非標準（パスを教えてください）
> **リポジトリ数**: 1〜3個 / 4〜10個 / 10個以上
> **移行先**: GitHub / GitHub Enterprise / Gitea（オンプレ）/ GitLab / その他
> **OS**: Windows（バージョン）/ Mac / Linux
> **現在のクライアント**: TortoiseSVN / svn CLI / その他

回答を受けたら、以下の移行方針を提示する:

```
移行方針サマリ
- SVN URL       : [入力値]
- レイアウト    : [標準 / 非標準]
- リポジトリ数  : [入力値]
- 移行先        : [入力値]
- 推奨アプローチ: [単体手順 / 一括スクリプト]
```

---

## Phase 1: 著者マッピングファイル生成

### 1.1 著者名抽出コマンドの生成

SVN URL と OS に応じてコマンドを生成する。

**PowerShell 版（Windows 標準推奨）:**
```powershell
# [SVN_URL] を実際の URL に置き換えて実行
svn log --quiet [SVN_URL] |
    Select-String "^r\d+" |
    ForEach-Object { $_ -replace "^r\d+ \| (.+?) \|.*$", '$1' } |
    Sort-Object -Unique |
    Out-File -FilePath authors_raw.txt -Encoding UTF8

Get-Content authors_raw.txt
```

**Bash 版（Git Bash / Mac / Linux）:**
```bash
svn log --quiet [SVN_URL] \
  | grep -E "^r[0-9]+" \
  | awk -F '|' '{print $2}' \
  | sort -u \
  > authors_raw.txt

cat authors_raw.txt
```

**複数リポジトリ一括抽出（PowerShell）:**
```powershell
$repos = @([各SVN URLをカンマ区切りで])
$allAuthors = @()
foreach ($url in $repos) {
    $allAuthors += svn log --quiet $url |
        Select-String "^r\d+" |
        ForEach-Object { $_ -replace "^r\d+ \| (.+?) \|.*$", '$1' }
}
$allAuthors | Sort-Object -Unique | Out-File authors_raw.txt -Encoding UTF8
```

### 1.2 authors.txt 作成テンプレートの生成

ユーザーが抽出した著者リストを貼り付けたら、以下の形式で `authors.txt` を生成する:

```
# SVNユーザー名 = Git表示名 <メールアドレス>
tanaka = 田中 太郎 <tanaka@example.co.jp>
suzuki = 鈴木 花子 <suzuki@example.co.jp>
# ※ 退職者・不明アカウントも必ず記載（空行にするとエラー）
old_user = 旧ユーザー <noreply@example.co.jp>
```

**注意事項を必ず伝える:**
- SVN ユーザー名は大文字小文字区別あり（完全一致）
- 全著者の行が必要（1行でも欠けると clone が停止する）
- 退職者・サービスアカウントも `noreply@` 等で記載する
- ファイルは UTF-8（BOM なし）で保存する

---

## Phase 2: git-svn clone コマンド生成

Phase 0 の回答に基づいて clone コマンドを生成する。

### 2.1 標準レイアウト（trunk/branches/tags）

```bash
# 作業ディレクトリに移動
cd [作業ディレクトリ]

git svn clone \
  --stdlayout \
  --authors-file=[authors.txt のフルパス] \
  --no-metadata \
  [SVN_URL] \
  [PROJECT_NAME]-git

cd [PROJECT_NAME]-git
```

### 2.2 非標準レイアウト

```bash
git svn clone \
  --trunk=[trunk相当のパス] \
  --branches=[branches相当のパス] \
  --tags=[tags相当のパス] \
  --authors-file=[authors.txt のフルパス] \
  --no-metadata \
  [SVN_URL] \
  [PROJECT_NAME]-git
```

### 2.3 複数リポジトリ一括処理（repos.csv → PowerShell スクリプト）

複数リポジトリの場合、以下の `repos.csv` 形式と一括スクリプトを生成する:

```csv
svn_url,project_name,layout
[SVN_URL_1],[PROJECT_1],standard
[SVN_URL_2],[PROJECT_2],standard
```

一括スクリプトは Phase 3 で合わせて生成する。

**大規模リポジトリの注意事項（10,000 リビジョン以上）:**
- clone には数時間〜1日以上かかる場合がある
- 途中停止した場合は `git svn fetch` で再開できる（やり直し不要）
- 夜間実行を推奨

---

## Phase 3: ブランチ・タグ変換スクリプト生成

clone 完了後に実行するスクリプトを生成する。

### 3.1 タグ変換

**Bash 版:**
```bash
git for-each-ref refs/remotes/origin/tags \
  --format='%(refname:short)' | \
  while read tag; do
    git tag "${tag#origin/tags/}" "refs/remotes/$tag"
    echo "タグ作成: ${tag#origin/tags/}"
  done
```

**PowerShell 版:**
```powershell
git for-each-ref refs/remotes/origin/tags --format='%(refname:short)' |
    ForEach-Object {
        $tagName = $_ -replace "^origin/tags/", ""
        git tag $tagName "refs/remotes/$_"
        Write-Host "タグ作成: $tagName"
    }
```

### 3.2 ブランチ変換

**Bash 版:**
```bash
for branch in $(git branch -r | grep -v '@' | grep -v 'trunk' | grep -v 'HEAD'); do
    branch_name="${branch##*/}"
    git branch "$branch_name" "$branch"
    echo "ブランチ作成: $branch_name"
done
```

**PowerShell 版:**
```powershell
git branch -r |
    Where-Object { $_ -notmatch "@" -and $_ -notmatch "trunk" -and $_ -notmatch "HEAD" } |
    ForEach-Object {
        $remoteBranch = $_.Trim()
        $branchName = $remoteBranch -replace "^.+/", ""
        git branch $branchName $remoteBranch
        Write-Host "ブランチ作成: $branchName"
    }
```

### 3.3 trunk を main にリネーム

```bash
git branch -m master main
# または（master が存在しない場合）
git checkout -b main refs/remotes/origin/trunk
```

### 3.4 複数リポジトリ一括処理スクリプト（PowerShell）

複数リポジトリの場合、clone → ブランチ変換 → push までを自動化した
`migrate_all.ps1` を生成する。ユーザーの環境（Git サーバ URL / Org 名）を
Phase 0 の回答から埋め込む。

---

## Phase 4: .gitignore 変換 + リモート push コマンド生成

### 4.1 .gitignore 生成

```bash
# SVN の svn:ignore を .gitignore に変換
git svn show-ignore > .gitignore

# Windows / Visual Studio の一般的な除外設定を追記（必要に応じて）
git add .gitignore
git commit -m "chore: SVNのsvn:ignoreを.gitignoreに変換"
```

追記推奨パターンをプロジェクト種別（.NET / Java / Node.js）に応じて提示する。

### 4.2 リモート push コマンド

Phase 0 で選択した移行先（GitHub / Gitea）に応じて URL を生成する。

```bash
# GitHub の場合
git remote add origin https://github.com/[ORG]/[PROJECT].git

# Gitea の場合
git remote add origin https://[GITEA_HOST]/[ORG]/[PROJECT].git

# 接続確認
git ls-remote origin

# プッシュ
git push -u origin main
git push --all origin
git push --tags origin
```

---

## Phase 5: チーム向け移行案内文・FAQ 生成

移行完了後にチームに配布する案内文を生成する。

### 5.1 移行完了通知メール

以下のテンプレートにプロジェクト名・URL・日付を埋め込んで生成する:

```
件名: 【完了】[プロジェクト名] SVN → Git 移行完了のご連絡

[プロジェクト名] の Git 移行が完了しました。

■ 新しいリポジトリ URL
  [GIT_URL]

■ 取得方法
  git clone [GIT_URL]
  または GitHub Desktop でクローン

■ SVN リポジトリの扱い
  [移行日] より SVN は読み取り専用になりました。
  新しいコミットは Git リポジトリで行ってください。

■ GitHub Desktop の導入手順
  [URL / 添付の手順書を参照]

■ 質問・相談: [担当者名・連絡先]
```

### 5.2 FAQ 生成

チームの規模・SVN 使用歴に応じて、よくある質問のリストを選定・カスタマイズして提示する。
最低限含めるべき質問:

1. git push したら何が起きますか？
2. 間違えて main に直接 commit しました
3. 変更を取得するコマンドは？（SVN update 相当）
4. コミットログに自分の名前が出ない
5. conflict が発生しました

---

## 禁止事項

- **移行前の SVN 書き込みロック確認を省略しない**: 移行中に SVN へのコミットが
  続くと履歴の乖離が生まれる。必ず「移行中は SVN への新規コミット禁止」を
  チームに周知してから作業開始する
- **空でないリポジトリへの clone & push を --force でごまかさない**:
  移行先は必ず「空のリポジトリ」として作成する
- **authors.txt の不完全な状態で clone を開始しない**: 途中停止のリスクがある。
  事前に全著者を抽出・確認してから開始する
- **工数の根拠なし見積もり禁止**: 「だいたい1日」などの根拠なし見積もりを出さない。
  リポジトリのリビジョン数が判明した時点で見積もる
