---
title: "【Mac】初期セットアップ"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mac"]
published: true
published_at: 2023-09-19 08:00
---

新しい Mac 購入後の初期セットアップを行うにあたり、以外と調べる作業が多かったので、記録として残しておきます。

## システム設定

左上のリンゴマーク(=アップルメニュー) ->「システム設定」と選択した時に表示される画面で設定していきます。
OS のバージョンによって設定画面が変わる可能性があるため、設定項目の場所までは詳しくは書かないですが大体わかると思います。
もし設定箇所がわからない場合は「mac 修飾キー 設定」などでググって調べてください。

### キーボード

- 修飾キーの設定：`Caps Lockキー を control に変更`
  - 入力中のカーソル移動のショートカットが使いやすくなり、コーディング時の操作性が格段に上がるのでおすすめです。
- ライブ変換：`OFF に変更`
  - 文字入力中に勝手に変換されてしまうのが嫌な方は OFF にしておくことをおすすめします。

### マウス

- 軌跡の速さ：`最大値の1つ下に変更`
  - 自身のお好みで調整してください。
- ナチュラルなスクロール：`(一旦)ON のまま`
  - 自身のお好みで調整してください。
  - 注意点として、この設定はトラックパッド側の設定と連動しているため「マウス側は OFF で トラックパッド側は ON」のような設定はできない。
  - 個人的に「マウスは OFF で トラックパッドは ON」にしたかったため、サードパーティー製ツールを利用して設定するようにした。（後述するが「ロジクール MX Master 3」を利用していたので「Logicool Options」を利用）

### トラックパッド

- 軌跡の速さ：`マウスと同じ速さに設定`
  - 自身のお好みで調整してください。
- ナチュラルなスクロール：`ON`
  - 自身のお好みで調整してください。
  - こちらの設定を変えるとマウス側の「ナチュラルなスクロール」の設定も連動して変更してしまいます。

### Dock

- Dock を自動的に表示／非表示：`ON`
  - 作業領域が広がるので ON にしておくのがおすすめです。
- 最近使ったアプリを Dock に表示：`OFF`
  - 自身のお好みで調整してください。

## Finder の設定

### サイドバーにホームディレクトリを表示する

Finder を開いて、メニューの 設定 -> サイドバー から自身のホームディレクトリにチェックを入れます。

### ホームディレクトリ直下に作業用のディレクトリを作成する

ホームディレクトリ直下に任意の名前で作業用のディレクトリ作成します。（僕は「dev」ディレクトリというのを作成しました）
この作業用のディレクトリ配下に作業に必要なものを置いたり、リポジトリをクローンしてきたりすることで、他の階層を汚さずに綺麗に運用することができます。

## 最低限必要なツールのインストール

### chrome

インストール後、デフォルトブラウザに設定する。
https://www.google.com/intl/ja/chrome/

### Raycast ※お好みで

これについては色々調べてもらえればと思いますが、とても万能なので spotlight をこちらに置き換えていきます。

1. spotlight の検索除外リストにハードディスクを追加
2. spotlight のショートカット無効化
3. raycast のホットキーを cmd+space に変更

https://www.raycast.com/

#### tips:

画面分割が便利なので設定しておくことをおすすめします。
:::details 設定方法

1. cmd+space で検索窓を開き General と検索する。
2. Extensions で half と検索する。
3. Bottom Half に cmd+Shift+↓ を設定
4. Left Half に cmd+Shift+← を設定
5. Right Half に cmd+Shift+→ を設定
6. Top Half に cmd+Shift+↑ を設定
   :::

### warp

iterm2 からこちらに乗り換えましたが使いやすく見た目もいいのでおすすめです。
Raycast で「左 ctrl のダブルタップ」で起動するようにホットキーに設定しています。
https://www.warp.dev/

### vscode

とりあえずインストールだけしてきます。
https://code.visualstudio.com/

### Homebrew

公式サイトの手順に沿ってインストールする。
https://brew.sh/

### Homebrew で git 管理を行うように変更

こちらの記事を参考にして設定する。
`.zshrc`に記載するパスだけ、記載されてる内容だとうまく行かなかったので`export PATH=/usr/local/bin/git:$PATH`を設定しました。
https://acokikoy.hatenablog.com/entry/2021/03/10/133558

## git の初期設定

以下を参照ください。
https://zenn.dev/hyzsa/articles/20230919-git-initial-setup

## Docker のインストール

以下の「事前準備」に記載してあります。
https://zenn.dev/hyzsa/articles/20230906-build-docker-rails7-postgresql

## 以下は必要な方のみ対応してください

### 1password

インストール後、chrome 拡張機能にも追加しておきます。
https://1password.com/jp

### brave

youtube で広告が表示されなくなるので便利です。
https://brave.com/ja/

### Logicool Options

ロジクール MX Master 3 を利用しているのでインストール

- スクロールの方向：標準
- サムホイールの方向：反転
- ※ポインタの速度は、Mac の設定で制御しておきたいため変えないこと

https://www.logicool.co.jp/ja-jp/software/options.html

### Homebrew で node brew と node.js のインストール

以下を参考にインストールを行う。
node.js は、zenn CLI を使うために Node.js 14 以上をインスールする必要があるので、とりあえず安定版をインストールしておく。
https://fromscratch-y.work/blog/programming/mac-nodejs-install/#sec1

## その他

### 画面のチラつき

ディスプレイの表示がチラつく現象が発生しました。
mac の初期不良を疑いましたが、以下の記事を参考にして、モニターのフレッシュレートを 144Hz→60Hz に変更したら改善しました。
https://outlook.aptrust.net/fix-mac-external-display-flikering/

### Mac の表示名を短いものに変える

こちらの記事を参考にしてコンピューター名を変更しておきました。
https://qiita.com/newt0/items/80164665e7e83ec7a669
