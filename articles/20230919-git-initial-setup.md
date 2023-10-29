---
title: "【Git】初期設定とSSH接続設定"
emoji: "📖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git", "github"]
published: false
published_at: 2023-09-19 08:00
---

## 事前準備

- 自身の mac に Git をインストールしておいてください。

```sh
git --version
# git version x.xx.x と表示されたらインストール済みです。
# versionが表示されずにデベロッパーツールのインストールが求められた場合はインストールを行なってください。
# インストール後に、再度コマンドを実行するとversionが表示されるはずです。
```

- Github に登録しておいてください。

https://github.co.jp/

## Git の初期設定

### ユーザー情報の設定

Github に登録した「ユーザー名」と「メールアドレス」を設定します。

```sh
git config --global user.name "ユーザー名"
```

```sh
git config --global user.email "メールアドレス"
```

## SSH 接続設定

### GitHub 用の SSH キーを生成

```sh
ssh-keygen -t ed25519 -N "" -f ~/.ssh/github
```

### GitHub に SSH キーを登録

1. クリップボードに公開鍵をコピーしておきます。

```sh
pbcopy < ~/.ssh/github.pub
```

2. GitHub の SSHkey 設定画面を開きます。

https://github.com/settings/keys

3. [New SSH Key]を選択します。
   ![](/images/20230919-git-initial-setup/New-SSH-key.png)

4. 必要な情報を入力していきます。
   ① 任意の名前を入力（任意だがこの鍵と紐づく PC がわかるようにしておくのがオススメ。PC 名とかが無難）
   ②「Authentication key」を選択
   ③ コピーした公開鍵を貼り付ける
   ![](/images/20230919-git-initial-setup/Add-new-SSH-Key.png)

5. 最後に ④ を選択して鍵を登録します。

### .ssh/config ファイルの作成

```sh
vi ~/.ssh/config
```

以下を追記します。

```sh
Host github.com
  IdentityFile ~/.ssh/github
  User git
```

### SSH 接続確認

```sh
ssh -T github.com
```

以下メッセージが表示されたら接続完了です。

```sh
Hi xxxx! You've successfully authenticated, but GitHub does not provide shell access.
```
