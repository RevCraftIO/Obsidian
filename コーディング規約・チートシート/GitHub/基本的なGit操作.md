GitHubを効果的に使用するには、Gitの基本的な操作を理解することが重要です。ここでは、日常的に使用する基本的なGitコマンドと操作フローを解説します。

## 作業の基本フロー

Gitを使った一般的な作業の流れは次のようになります：

1. リポジトリをクローンまたはプル
2. ブランチを作成
3. ファイルを変更
4. 変更をステージング
5. 変更をコミット
6. リモートリポジトリにプッシュ

## 基本コマンド一覧

### リポジトリの取得と更新

```bash
# リポジトリをクローン
git clone <リポジトリURL>

# リモートの変更を取得するが、マージはしない
git fetch

# リモートの変更を取得してマージする
git pull

# 特定のブランチをプル
git pull origin <ブランチ名>
```

### ブランチ操作

```bash
# ブランチ一覧を表示
git branch

# 新しいブランチを作成
git branch <ブランチ名>

# ブランチを切り替え
git checkout <ブランチ名>

# ブランチ作成と切り替えを同時に行う
git checkout -b <ブランチ名>

# ブランチの削除
git branch -d <ブランチ名>

# リモートブランチを含むすべてのブランチを表示
git branch -a
```

### 変更の確認

```bash
# 変更されたファイルを確認
git status

# 変更内容の詳細を確認
git diff

# コミット間の差分を確認
git diff <コミットID1> <コミットID2>

# ステージングされた変更の差分を確認
git diff --staged
```

### 変更のステージングとコミット

```bash
# 特定のファイルをステージングに追加
git add <ファイル名>

# すべての変更をステージングに追加
git add .

# 変更をコミット
git commit -m "コミットメッセージ"

# ステージングとコミットを同時に行う（新規ファイル以外）
git commit -am "コミットメッセージ"
```

### リモートリポジトリとのやり取り

```bash
# 変更をリモートリポジトリにプッシュ
git push origin <ブランチ名>

# リモートブランチを追跡設定してプッシュ
git push -u origin <ブランチ名>

# リモートリポジトリ情報を表示
git remote -v

# リモートリポジトリを追加
git remote add <名前> <リポジトリURL>
```

### 履歴の確認

```bash
# コミット履歴を表示
git log

# コミット履歴をグラフ形式で表示
git log --graph --oneline --all

# 特定ファイルの変更履歴を表示
git log --follow <ファイル名>
```

### 変更の取り消し

```bash
# 直前のコミットの修正
git commit --amend

# ステージングした変更を取り消す（ファイルは変更したまま）
git reset <ファイル名>

# 変更を完全に取り消して特定のコミットに戻る
git reset --hard <コミットID>

# 特定のファイルを最後のコミット状態に戻す
git checkout -- <ファイル名>

# コミットを取り消す新しいコミットを作成（推奨）
git revert <コミットID>
```

## 実践的なワークフロー例

### 1. 新機能開発の基本フロー

```bash
# メインブランチを最新に更新
git checkout main
git pull

# 機能開発用の新しいブランチを作成
git checkout -b feature/new-feature

# ファイルを変更、追加など...

# 変更をコミット
git add .
git commit -m "新機能の実装: ○○機能"

# 開発中に定期的にメインブランチの変更を取り込む
git checkout main
git pull
git checkout feature/new-feature
git merge main

# 機能が完成したらリモートにプッシュ
git push -u origin feature/new-feature

# その後GitHub上でプルリクエストを作成
```

### 2. バグ修正の基本フロー

```bash
# バグ修正用ブランチを作成
git checkout -b fix/bug-description

# バグを修正...

# コミット
git add <修正したファイル>
git commit -m "Fix: バグの説明と修正内容"

# リモートにプッシュ
git push -u origin fix/bug-description
```

### 3. コンフリクト解決の流れ

マージ時にコンフリクト（競合）が発生した場合：

```bash
# マージでコンフリクトが発生
git merge main
# CONFLICT: Merge conflict in file.txt

# コンフリクトファイルを確認
git status

# ファイルを開いてコンフリクトを手動で解決
# <<<<<< HEAD
# あなたの変更
# =======
# 他の人の変更
# >>>>>>> main
# ↑このような部分を適切に編集

# 解決したファイルをステージングに追加
git add <解決したファイル>

# マージを完了
git commit
```

## .gitignoreの活用

プロジェクトに不要なファイルをGitの管理対象から除外するために、`.gitignore`ファイルを活用しましょう。

```
# 一般的な.gitignoreの例

# OSが生成するファイル
.DS_Store
Thumbs.db

# エディタの設定ファイル
.idea/
.vscode/

# ビルド成果物
/dist/
/build/
/out/

# 依存関係
/node_modules/
/vendor/

# ログファイル
*.log

# 環境設定ファイル
.env
.env.local
```

## Gitエイリアスで効率化

よく使うコマンドにエイリアス（別名）を設定すると作業効率が上がります。

```bash
# コミットのエイリアス
git config --global alias.ci "commit"

# ステータス確認のエイリアス
git config --global alias.st "status"

# ブランチ切り替えのエイリアス
git config --global alias.co "checkout"

# ログを見やすく表示するエイリアス
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset' --abbrev-commit --date=relative"
```

これらのエイリアスを設定すると、例えば `git st` と入力するだけで `git status` が実行できます。

## 次のステップ

Git操作の基本を理解したら、次は[[プルリクエストとコードレビュー]]を学んで、チーム開発の中核となるプロセスを理解しましょう。