---
title: "【Rails】rails歴1年の僕が「パーフェクト Ruby on Rails」で学べたこと（随時更新）"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rails"]
published: true
published_at: 2023-09-25 08:00
---

web エンジニア 1 年目の僕が「パーフェクト Ruby on Rails」で新しく学べた知識や、忘れかけてたことについて記録していきます。

https://gihyo.jp/book/2020/978-4-297-11462-6

## Rails の思想(p24)

- CoC(Convention over Configuration)
  - 「設定より規約」という意味
- DRY(Don't Repeat Yourself)
  - 「同じことを繰り返さない」という思想
- REST(Representational State Transfer)
  - Web アプリケーションの設計概念の一つ
- 自動テスト
  - rails では自動テストを行う文化を重要視している

## schema_migrations テーブルについて(p40)

migration の管理を行うテーブルです。
rails db:migrate を実行すると、まずこのテーブルを参照しにいき、テーブルに migration ID が記録されていない migration ファイルだけが実行されて、migration ID が記録されます。
そのため、rails db:migrate を複数回実行しても、重複して migration ファイルが実行されることはありません。

## ActiveRecord：クエリは実際にデータが必要になったタイミングで発行される(p56)

```ruby
books = Book.where('price > ?', 3000)
# まだクエリ発行されない
# ActiveRecord に対して Query Interface(where や limit など) が呼ばれると、ActiveRecord::Relation のインスタンス(=books)が生成される。
# この ActiveRecord::Relation のインスタンス(=books)に、どういうクエリを発行するかの情報が記録される。
books.count
# 実際にデータが必要になったのでクエリが発行される
```

この挙動については、以下の記事で詳しく解説してくれてます。
https://qiita.com/ykamez/items/0c81a33ec1b90219d541

## scope では検索条件にマッチしなかった場合に nil が返らない(p58)

検索条件にマッチしなかった場合、その scope を除外したクエリが発行されるため、必ず ActiveRecord::Relation が返る。
nil を返す必要がある場合はクラスメソッドとして定義する方が良い

### 検証

| id  | name   | published_on                | price |
| --- | ------ | --------------------------- | ----- |
| 1   | Book 1 | '2019-11-24 00:00:00 +0000' | 1000  |
| 2   | Book 2 | '2019-10-24 00:00:00 +0000' | 2000  |
| 3   | Book 3 | '2019-09-24 00:00:00 +0000' | 3000  |
| 4   | Book 4 | '2019-08-24 00:00:00 +0000' | 4000  |
| 5   | Book 5 | '2019-07-24 00:00:00 +0000' | 5000  |

```ruby
class Book < ApplicationRecord
  scope :costly, -> { where("price > ?", 3000) }
  scope :find_price, ->(price) { find_by(price: price) }
end
```

「3000 円以上の中から、6000 円の本を探す(`Book.costly.find_price(6000)`)」が 6000 円の本が見つからないので、「3000 円以上の本(`Book.costly`)」の結果が返る。

```ruby
irb(main):001:0> Book.costly.find_price(5000)
  Book Load (0.8ms)  SELECT "books".* FROM "books" WHERE (price > 3000) AND "books"."price" = $1 LIMIT $2  [["price", 5000], ["LIMIT", 1]]
=> #<Book id: 5, name: "Book 5", published_on: "2019-07-24", price: 5000, created_at: "2023-09-23 17:33:18", updated_at: "2023-09-23 17:33:18">
irb(main):002:0> Book.costly.find_price(6000)
  Book Load (1.1ms)  SELECT "books".* FROM "books" WHERE (price > 3000) AND "books"."price" = $1 LIMIT $2  [["price", 6000], ["LIMIT", 1]]
  Book Load (0.6ms)  SELECT "books".* FROM "books" WHERE (price > 3000) LIMIT $1  [["LIMIT", 11]]
=> #<ActiveRecord::Relation [#<Book id: 4, name: "Book 4", published_on: "2019-08-24", price: 4000, created_at: "2023-09-23 17:33:18", updated_at: "2023-09-23 17:33:18">, #<Book id: 5, name: "Book 5", published_on: "2019-07-24", price: 5000, created_at: "2023-09-23 17:33:18", updated_at: "2023-09-23 17:33:18">]>
```

一方でクラスメソッドとして定義した場合は、「3000 円以上の中から、6000 円の本を探す(`Book.costly.find_price(6000)`)」が 6000 円の本が見つからないので`nil`が返る。

```ruby
class Book < ApplicationRecord
  scope :costly, -> { where("price > ?", 3000) }

  def self.find_price(price)
    find_by(price: price)
  end
end
```

```ruby
irb(main):001:0> Book.costly.find_price(6000)
  Book Load (0.7ms)  SELECT "books".* FROM "books" WHERE (price > 3000) AND "books"."price" = $1 LIMIT $2  [["price", 6000], ["LIMIT", 1]]
=> nil
```
