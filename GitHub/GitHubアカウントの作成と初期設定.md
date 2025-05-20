## アカウント作成手順

1. [GitHub公式サイト](https://github.com/)にアクセスする
2. 「Sign up」ボタンをクリック
3. 以下の情報を入力:
    - メールアドレス
    - パスワード
    - ユーザー名（一意である必要があります）
4. メールアドレスの確認
5. アカウント種別の選択（無料か有料プラン）

## プロフィールの設定

アカウント作成後は、プロフィールを充実させることでより効果的にGitHubを活用できます。

### 設定すべき項目

- **プロフィール画像**: 認識しやすい画像を設定する
- **自己紹介文**: スキルや興味のある分野などを簡潔に記載
- **場所**: 国や地域（必須ではありません）
- **組織/会社**: 所属がある場合は記載
- **ウェブサイト**: ブログやポートフォリオサイトなど

### プロフィールREADME

GitHubでは自分のユーザー名と同じ名前のリポジトリを作成し、そこにREADME.mdファイルを置くことで、プロフィールページにカスタマイズされた内容を表示できます。

```
例: ユーザー名が「user123」の場合
リポジトリ名: user123
その中にREADME.mdファイルを作成
```

## SSHキーの設定

ローカル環境からGitHubへのセキュアな接続を確立するために、SSHキーの設定をおすすめします。

### SSHキー生成手順

1. ターミナル（コマンドプロンプト）を開く
2. 以下のコマンドを実行:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

3. 保存先とパスフレーズを設定（デフォルトでも可）
4. 公開キーをクリップボードにコピー:

**Mac**:

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

**Windows** (Git Bash):

```bash
cat ~/.ssh/id_ed25519.pub | clip
```

**Linux**:

```bash
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard
```

### GitHubへのSSHキー登録

1. GitHubにログイン
2. 右上のプロフィールアイコンをクリック → Settings
3. 左側のメニューから「SSH and GPG keys」を選択
4. 「New SSH key」ボタンをクリック
5. タイトルを入力（例: 「Work Laptop」）
6. 「Key」欄にコピーした公開キーを貼り付け
7. 「Add SSH key」ボタンをクリック

## 二要素認証の設定

アカウントのセキュリティを強化するために、二要素認証(2FA)の設定を強く推奨します。

### 設定手順

1. GitHubにログイン
2. Settings → Security → Two-factor authentication
3. 「Enable two-factor authentication」をクリック
4. 認証方法の選択:
    - スマートフォンアプリ（推奨）
    - SMSによる認証
5. 画面の指示に従って設定を完了
6. リカバリーコードを安全な場所に保存

## Gitのグローバル設定

ローカル環境でGitを使用する際の基本設定です。

```bash
# ユーザー名の設定
git config --global user.name "あなたの名前"

# メールアドレスの設定
git config --global user.email "your_email@example.com"

# デフォルトブランチ名の設定（最近のGitではmainが推奨されています）
git config --global init.defaultBranch main

# 設定内容の確認
git config --list
```

## GitHubデスクトップのインストール（オプション）

コマンドライン操作が不慣れな場合、GitHubデスクトップを使うとGUIでGit操作ができます。

1. [GitHub Desktop](https://desktop.github.com/)のウェブサイトにアクセス
2. お使いのOSに合わせたバージョンをダウンロード
3. インストーラーを実行してインストール
4. GitHubアカウントでログイン

## 次のステップ

アカウントと環境の初期設定が完了したら、次は[[リポジトリの作成と管理]]を学びましょう。