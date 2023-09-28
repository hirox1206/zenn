---
title: "【Docker】rails7 + PostgreSQLのdocker開発環境を構築する"
emoji: "🛠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker", "rails"]
published: true
published_at: 2023-09-06 08:00
---

## はじめに

この記事では、Alpine Linux ベースの Ruby イメージを使って、rails7 + PostgreSQL の docker 開発環境を構築していきます。

## 対象

- docker で rails が動く環境をサクッと作りたい方

## 環境

- macOS
- Docker version 20.10.14
- docker-compose version 1.29.2
- ruby 3.2.2
- Rails 7.0.7.2
- PostgreSQL 15.4

## 事前準備

- 自身の PC に`docker`と`docker-compose`をインストールしておいてください

https://matsuand.github.io/docs.docker.jp.onthefly/desktop/mac/install/

> Docker Desktop for Mac には他の Docker アプリとともに Compose が含まれています。 したがって Mac ユーザーは Compose を個別にインストールする必要はありません。
> _（引用元：https://matsuand.github.io/docs.docker.jp.onthefly/compose/install/ ）_

## 必要なファイルの準備

任意の場所にプロジェクト管理用のフォルダを作成します。

```sh
mkdir docker-rails  # フォルダ作成（フォルダ名は任意です）
cd docker-rails     # 作成したフォルダに移動する
```

ここに以下のような構成を作っていきます。

```sh
.
├── docker
│   └── rails
│       └── Dockerfile
├── docker-compose.yml
└── src
    ├── Gemfile
    └── Gemfile.lock

4 directories, 4 files
```

### ■ Dockerfile

```sh
mkdir -p docker/rails
touch docker/rails/Dockerfile
```

```docker:Dockerfile
FROM ruby:3.2-alpine

RUN apk update && \
    apk add --no-cache gcompat && \
    apk add --no-cache linux-headers libxml2-dev make gcc libc-dev nodejs tzdata postgresql-dev postgresql git bash && \
    apk add --virtual build-packages --no-cache build-base curl-dev

RUN mkdir /myapp
WORKDIR /myapp
ADD src/Gemfile /myapp/Gemfile
ADD src/Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
RUN apk del build-packages
ADD . /myapp
EXPOSE 4000
CMD ["rails", "server", "-b", "0.0.0.0"]
```

### ■ Gemfile, Gemfile.lock

```sh
mkdir src
touch src/{Gemfile,Gemfile.lock}
```

```ruby:Gemfile
source 'https://rubygems.org'
gem 'rails', '~>7.0.3'
```

```ruby:Gemfile.lock
# Gemfile.lockは空のままでOKです
```

### ■ docker-compose.yml

```sh
touch docker-compose.yml
```

```yml:docker-compose.yml
version: '3.9'
services:
  db:
    image: postgres
    volumes:
      - ./src/tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password
  web:
    build:
      context: .
      dockerfile: ./docker/rails/Dockerfile
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 4000 -b '0.0.0.0'"
    stdin_open: true
    tty: true
    volumes:
      - ./src:/myapp
    ports:
      - "4000:4000"
    depends_on:
      - db
```

## 環境を立ち上げていく

### ■ rails new

Rails のアプリケーションを作成します。
以下コマンドを実行すると、`src`フォルダ配下に、Rails アプリケーションに最低限必要なフォルダやファイルが自動的に生成されます。

```sh
docker compose run web rails new . --force --database=postgresql
```

`rails new`したことで`Gemfile`が更新されたので`bundle install`を行います。

```sh
docker compose build
```

### ■ DB 接続設定

rails が DB に接続するための設定を`src/config/database.yml`に記述します。
デフォルトで記述がありますが、以下で上書きしてください。

```ruby:database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password: password
  pool: 5

development:
  <<: *default
  database: myapp_development

test:
  <<: *default
  database: myapp_test
```

### ■ 起動

DB を作成します。

```sh
docker compose run web rails db:create
```

コンテナを起動します。

```sh
docker compose up -d
```

ブラウザで http://localhost:4000/ にアクセスして、Rails が立ち上がっていれば成功です！
