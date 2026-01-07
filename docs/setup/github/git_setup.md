# Git セットアップ手順

## GitHubリポジトリの作成とメンバー招待

### リポジトリの作成

1. GitHubにログインし、右上の「+」アイコンをクリック
2. 「New repository」を選択
3. リポジトリ情報を入力：
   - Repository name: プロジェクト名を入力
   - Description: プロジェクトの説明（任意）
   - Public/Private: 公開範囲を選択
   - Initialize this repository with: 必要に応じてREADME、.gitignore、Licenseを追加
4. 「Create repository」をクリック

### メンバーの招待

1. リポジトリページで「Settings」タブをクリック
2. 左サイドバーから「Collaborators」または「Manage access」を選択
3. 「Add people」または「Invite a collaborator」ボタンをクリック
4. 招待するユーザーのGitHubユーザー名またはメールアドレスを入力
5. 権限レベル（Read, Write, Adminなど）を選択
6. 「Add [username] to this repository」をクリック
7. 招待されたユーザーにメール通知が送信され、承認後にアクセス可能になる

## SSH接続の設定

### default設定version

```bash
# SSH鍵の生成（パスワードなし）
ssh-keygen -t ed25519 -f ~/.ssh/[YourFavoriteKeyName] -N "" -C "your_email@example.com"

# 公開鍵をコピー
cat ~/.ssh/[YourFavoriteKeyName].pub

# GitHubに公開鍵を登録
# 1. GitHub Settings > SSH and GPG keys > New SSH key
# 2. タイトルを入力し、公開鍵を貼り付けて保存
```

#### SSH設定ファイルの編集

`~/.ssh/config`に以下を追加：

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/[YourFavoriteKeyName]
    IdentitiesOnly yes
```

#### 疎通確認

```bash
# SSH接続テスト
ssh -T git@github.com

# 成功すると以下のようなメッセージが表示される
# Hi [username]! You've successfully authenticated, but GitHub does not provide shell access.
```

#### リポジトリのクローン

```bash
# SSH経由でクローン
git clone git@[GitUserName]/[GitRepositoryName].git
```

### Host変更version

```bash
# SSH鍵の生成（パスワードなし）
ssh-keygen -t ed25519 -f ~/.ssh/github-[HostAlias] -N "" -C "your_email@example.com"

# 公開鍵をコピー
cat ~/.ssh/github-[HostAlias].pub

# GitHubに公開鍵を登録
# 1. GitHub Settings > SSH and GPG keys > New SSH key
# 2. タイトルを入力し、公開鍵を貼り付けて保存
```

#### SSH設定ファイルの編集

`~/.ssh/config`に以下を追加：

```
Host github-[HostAlias]
    HostName github.com
    User git
    IdentityFile ~/.ssh/github-[HostAlias]
    IdentitiesOnly yes
```

他のconfigと重複しないように[Hostname]を変更するversionもあります：
```
Host github-[HostAlias]
    HostName github.com
    User git
    IdentityFile ~/.ssh/github-[HostAlias]
    IdentitiesOnly yes
```

#### 疎通確認

```bash
# SSH接続テスト
ssh -T github-[HostAlias]

# 成功すると以下のようなメッセージが表示される
# Hi [username]! You've successfully authenticated, but GitHub does not provide shell access.
```

#### リポジトリのクローン

```bash
# SSH経由でクローン
git clone github-[HostAlias]:[GitUserName]/[GitRepositoryName].git
```
