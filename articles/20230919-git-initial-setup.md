---
title: "【Git】初期設定とSSH接続設定"
emoji: "📖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git", "github"]
published: true
published_at: 2023-09-19 08:00
---

## 事前準備
- 自身のmacにGitをインストールしておいてください。
```sh
git --version
# git version x.xx.x と表示されたらインストール済みです。
# versionが表示されずにデベロッパーツールのインストールが求められた場合はインストールを行なってください。
# インストール後に、再度コマンドを実行するとversionが表示されるはずです。
```

- Githubに登録しておいてください。

https://github.co.jp/

## Gitの初期設定
### ユーザー情報の設定
Githubに登録した「ユーザー名」と「メールアドレス」を設定します。
```sh
git config --global user.name "ユーザー名"
```
```sh
git config --global user.email "メールアドレス"
```

## SSH接続設定

### GitHub用のSSHキーを生成
```sh
ssh-keygen -t ed25519 -N "" -f ~/.ssh/github
```

### GitHubにSSHキーを登録
1. クリップボードに公開鍵をコピーしておきます。
```sh
pbcopy < ~/.ssh/github.pub
```

2. GitHubのSSHkey設定画面を開きます。

https://github.com/settings/keys

3. [New SSH Key]を選択します。
![](/images/20230919-git-initial-setup/New-SSH-key.png)

4. 必要な情報を入力していきます。
①任意の名前を入力（任意だがこの鍵と紐づくPCがわかるようにしておくのがオススメ。PC名とかが無難）
②「Authentication key」を選択
③コピーした公開鍵を貼り付ける
![](/images/20230919-git-initial-setup/Add-new-SSH-Key.png)

5. 最後に④を選択して鍵を登録します。

### .ssh/configファイルの作成
```sh
vi ~/.ssh/config
```
以下を追記します。
```sh
Host github.com
  IdentityFile ~/.ssh/github
  User git
```

### SSH接続確認
```sh
ssh -T github.com
```

以下メッセージが表示されたら接続完了です。
```sh
Hi xxxx! You've successfully authenticated, but GitHub does not provide shell access.
```
