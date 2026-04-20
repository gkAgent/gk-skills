# SVN → Git 移行キット

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/migration-svn-git` で自動起動します（貼り付け不要）。

---

以下の指示に従って、SVN リポジトリの Git 移行を支援してください。フェーズを順番に進め、スキップしないでください。

---

## Phase 0: SVN 環境を確認する

まず以下を質問してください:

> 移行環境を教えてください。
>
> **SVN サーバ**: URL（例: http://svn-server/repos/PROJECT）/ VisualSVN Server / その他
> **レイアウト**: 標準（trunk/branches/tags）/ 非標準（パスを教えてください）
> **リポジトリ数**: 1〜3個 / 4〜10個 / 10個以上
> **移行先**: GitHub / GitHub Enterprise / Gitea（オンプレ）/ GitLab / その他
> **OS**: Windows（バージョン）/ Mac / Linux
> **現在のクライアント**: TortoiseSVN / svn CLI / その他

回答を受けたら、移行方針のサマリを提示してください。

---

## Phase 1: 著者マッピングファイルを生成する

### 1.1 著者名抽出コマンドを生成する

OS に応じて以下のコマンドを生成してください。

**PowerShell 版（Windows）:**
```powershell
svn log --quiet [SVN_URL] |
    Select-String "^r\d+" |
    ForEach-Object { $_ -replace "^r\d+ \| (.+?) \|.*$", '$1' } |
    Sort-Object -Unique |
    Out-File -FilePath authors_raw.txt -Encoding UTF8
```

**Bash 版（Git Bash / Mac / Linux）:**
```bash
svn log --quiet [SVN_URL] \
  | grep -E "^r[0-9]+" \
  | awk -F '|' '{print $2}' \
  | sort -u \
  > authors_raw.txt
```

複数リポジトリの場合は一括抽出スクリプトも生成してください。

### 1.2 authors.txt テンプレートを生成する

ユーザーが著者リストを貼り付けたら、以下の形式で authors.txt を生成してください:

```
# SVNユーザー名 = Git表示名 <メールアドレス>
tanaka = 田中 太郎 <tanaka@example.co.jp>
suzuki = 鈴木 花子 <suzuki@example.co.jp>
old_user = 旧ユーザー <noreply@example.co.jp>
```

以下を必ず伝えてください:
- 全著者の行が必要（1行でも欠けると clone が停止する）
- 退職者・サービスアカウントも noreply@ 等で記載する
- ファイルは UTF-8（BOM なし）で保存する

---

## Phase 2: git-svn clone コマンドを生成する

Phase 0 の回答に基づいて clone コマンドを生成してください。

**標準レイアウト（trunk/branches/tags）の場合:**
```bash
git svn clone \
  --stdlayout \
  --authors-file=[authors.txt のフルパス] \
  --no-metadata \
  [SVN_URL] \
  [PROJECT_NAME]-git
```

**非標準レイアウトの場合:**
```bash
git svn clone \
  --trunk=[trunk相当パス] \
  --branches=[branches相当パス] \
  --tags=[tags相当パス] \
  --authors-file=[authors.txt のフルパス] \
  --no-metadata \
  [SVN_URL] \
  [PROJECT_NAME]-git
```

複数リポジトリの場合は repos.csv と PowerShell 一括スクリプトも生成してください。

大規模リポジトリ（10,000 リビジョン以上）の場合は途中停止時の再開手順も示してください:
```bash
cd [PROJECT_NAME]-git
git svn fetch  # 最後のリビジョンから自動再開
```

---

## Phase 3: ブランチ・タグ変換スクリプトを生成する

clone 完了後に実行するスクリプトを Bash 版と PowerShell 版の両方で生成してください。

**タグ変換（Bash）:**
```bash
git for-each-ref refs/remotes/origin/tags \
  --format='%(refname:short)' | \
  while read tag; do
    git tag "${tag#origin/tags/}" "refs/remotes/$tag"
  done
```

**タグ変換（PowerShell）:**
```powershell
git for-each-ref refs/remotes/origin/tags --format='%(refname:short)' |
    ForEach-Object {
        $tagName = $_ -replace "^origin/tags/", ""
        git tag $tagName "refs/remotes/$_"
    }
```

**ブランチ変換（Bash）:**
```bash
for branch in $(git branch -r | grep -v '@' | grep -v 'trunk' | grep -v 'HEAD'); do
    git branch "${branch##*/}" "$branch"
done
```

**ブランチ変換（PowerShell）:**
```powershell
git branch -r |
    Where-Object { $_ -notmatch "@" -and $_ -notmatch "trunk" -and $_ -notmatch "HEAD" } |
    ForEach-Object {
        $b = $_.Trim()
        git branch ($b -replace "^.+/", "") $b
    }
```

trunk を main にリネームするコマンドも生成してください:
```bash
git branch -m master main
```

---

## Phase 4: .gitignore 変換とリモート push コマンドを生成する

**.gitignore 生成:**
```bash
git svn show-ignore > .gitignore
git add .gitignore
git commit -m "chore: SVNのsvn:ignoreを.gitignoreに変換"
```

プロジェクト種別（.NET / Java / Node.js）に応じた追記推奨パターンも提示してください。

**リモート push（移行先に応じて URL を生成）:**
```bash
git remote add origin https://[GIT_SERVER]/[ORG]/[PROJECT].git
git ls-remote origin  # 接続確認
git push -u origin main
git push --all origin
git push --tags origin
```

---

## Phase 5: チーム向け移行案内文と FAQ を生成する

### 移行完了通知メールを生成する

以下のテンプレートにプロジェクト情報を埋め込んでください:

```
件名: 【完了】[プロジェクト名] SVN → Git 移行完了のご連絡

[プロジェクト名] の Git 移行が完了しました。

■ 新しいリポジトリ URL
  [GIT_URL]

■ SVN リポジトリの扱い
  [移行日] より SVN は読み取り専用になりました。

■ 取得方法
  git clone [GIT_URL]
  または GitHub Desktop でクローン

■ 質問・相談: [担当者名・連絡先]
```

### FAQ を生成する

以下の質問に対する回答を生成してください:
1. git push したら何が起きますか？
2. 間違えて main ブランチに直接 commit しました
3. SVN の update に相当する操作は何ですか？
4. コミットログに自分の名前が正しく出ません
5. conflict（競合）が発生しました
6. 大きなファイル（Excel・画像など）はコミットしてもいいですか？

---

## 禁止事項

- **移行中に SVN への書き込みが続いている状態で作業しない**: チームへの周知と SVN ロックを先に行う
- **空でないリポジトリへのプッシュを --force で解決しない**: 移行先は必ず空で作成する
- **authors.txt が不完全な状態で clone を開始しない**: 全著者の確認を先に完了させる
- **根拠なし工数見積もりを出さない**: リビジョン数が判明した時点で見積もる
