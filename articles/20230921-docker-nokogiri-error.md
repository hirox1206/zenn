---
title: "【docker】nokogiri 関連のエラーを解決する方法"
emoji: "🧩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker", "rails"]
published: false
published_at: 2023-09-21 08:00
---

## エラーの内容

docker イメージのビルド時に`It looks like you're trying to use Nokogiri as a precompiled native gem`というエラーが発生しました。
どうやら、Apple シリコン搭載の Mac(M1, M2)で発生するエラーのようです。

```sh
ERROR: It looks like you're trying to use Nokogiri as a precompiled native gem on a system with glibc < 2.17:
  /lib/aarch64-linux-gnu/libm.so.6: version `GLIBC_2.29' not found (required by /usr/local/bundle/gems/nokogiri-1.13.10-aarch64-linux/lib/nokogiri/2.6/nokogiri.so) - /usr/local/bundle/gems/nokogiri-1.13.10-aarch64-linux/lib/nokogiri/2.6/nokogiri.so
  If that's the case, then please install Nokogiri via the `ruby` platform gem:
      gem install nokogiri --platform=ruby
  or:
      bundle config set force_ruby_platform true
  Please visit https://nokogiri.org/tutorials/installing_nokogiri.html for more help.
```

## 解決方法

Dockerfile の`bundle install`前に以下を追記してあげることで解決できます。
Bundler のバージョンは`Gemfile.lock`の最下部にある`BUNDLED WITH`で確認することができます。

### Bundler のバージョンが`2.1`以上の場合

```sh
RUN bundle config set force_ruby_platform true
RUN bundle install
```

### Bundler のバージョンが`2.1`より下の場合

```sh
RUN bundle config force_ruby_platform true
RUN bundle install
```
