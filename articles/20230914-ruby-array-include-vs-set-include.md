---
title: "【Ruby】Array#include?を高速化する方法"
emoji: "💎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ruby"]
published: false
---
## 結論
レシーバとなる要素を`to_set`メソッドで`Set`オブジェクトに変換し、`include?`を使うと高速に判定できます。

`Array#include?`は検索アルゴリズムの線形探索(O(n))を行うため、データ量(=要素数)が多くなるほど処理時間が増加します。一方、`Set#include?`は内部的にハッシュを使用しているため、処理時間はデータ量(=要素数)に影響されず常に一定(O(1))です。

```ruby
array = [1, 2, 3, 4, 5]
set = array.to_set

array.include?(4) # O(n)
set.include?(4) # O(1) ※こちらのほうが高速
```

| メソッド | 計算量(O記法) | 特徴 |
| ------ | ------------ | ---- |
| `Array#include?` | O(n) | ・検索アルゴリズムの線型探索にあたる <br>・データ量が多くなるほど処理に時間がかかる |
| `Set#include?` | O(1) | ・検索アルゴリズムのハッシュ法にあたる <br> ・処理時間はデータ量に関係なく一定 |

## 処理の深堀り
### Array#include?
**配列の先頭から順番に引数の値と比較**し、一致する要素を見つけた時点でtrueを返します。

```ruby
array = [1, 2, 3, 4, 5]
array.include?(4) # 1→2→3→4 と先頭から順番に確認し、4を見つけた時点で true
```

:::details Array#include?の内部実装
https://github.com/ruby/ruby/blob/a98209b8a70345714ac5f3028e0591f3ee50bba7/array.c#L5212-L5225

内部実装を見てみると、配列の先頭から順番に比較していることがわかります。そのため、対象の要素が先頭から遠ければ遠いほど、一致するまでに時間がかかります。

また、対象の要素が配列内に存在していない場合は、途中で一致することがないので、配列の先頭から末尾までを全て比較することになります。
:::

### Set#include?
`Set`オブジェクトは内部的にハッシュを保持しており、要素の存在確認が高速です。

```ruby
set = [1, 2, 3, 4, 5].to_set
# setオブジェクトは内部的に次のようなハッシュを保持する。（デフォルト値: false）
# @hash = { 1 => true, 2 => true, 3 => true, 4 => true, 5 => true }

set.include?(4) # 直接 @hash[4] を参照するので高速
```

:::details to_setの内部実装
### ①Enumerable#to_set
https://github.com/ruby/ruby/blob/4655d2108ef14e66f64496f9029f65ba2302d9ea/prelude.rb#L28-L30
`to_set`を呼び出すと、まずは`Set.new`で`Set`オブジェクトを生成します。

### ②Set#initialize
https://github.com/ruby/ruby/blob/a98209b8a70345714ac5f3028e0591f3ee50bba7/lib/set.rb#L243-L253
オブジェクトが生成されるとコンストラクタが呼び出されます。

ここで、`@hash(デフォルト値: false)`を生成しています。そして`block`は渡していないので、`merge(enum)`が実行されます。

### ③Set#merge
https://github.com/ruby/ruby/blob/a98209b8a70345714ac5f3028e0591f3ee50bba7/lib/set.rb#L603-L613
引数で渡された`enum`を可変長引数として受け取ります。`enum`は`to_set`呼び出し時のレシーバの要素(ex. `[1, 2, 3, 4, 5]`)なので、可変長引数として受け取ることで`[[1, 2, 3, 4, 5]]`のような2次元配列になります。

そのため、`do_with_enum(enum) { |o| add(o) }`側の処理が実行されます。

### ④Set#do_with_enum
https://github.com/ruby/ruby/blob/a98209b8a70345714ac5f3028e0591f3ee50bba7/lib/set.rb#L272-L281

引数の`enum`は`[1, 2, 3, 4, 5]`なので配列です。配列は`each_entry`メソッドを保持しているので`enum.each_entry(&block) if block`が実行されます。

`each_entry`メソッドは、各要素にブロックの処理を一度ずつ適用するメソッドです。そのため`[1, 2, 3, 4, 5]`の各要素に対して引数で渡されたブロックの処理(=`{ |o| add(o) }`)が実行されます。

#### ⑤Set#add
https://github.com/ruby/ruby/blob/a98209b8a70345714ac5f3028e0591f3ee50bba7/lib/set.rb#L519-L522
引数の`o`は、`[1, 2, 3, 4, 5]`の各要素で、各要素を`@hash`のキーとして格納します。

そのため、`[1, 2, 3, 4, 5].to_set`を行うと内部的に以下のようなハッシュが生成されて保持されます。
```ruby
@hash = { 1 => true, 2 => true, 3 => true, 4 => true, 5 => true }
```
:::

:::details Set#include?の内部実装
https://github.com/ruby/ruby/blob/a98209b8a70345714ac5f3028e0591f3ee50bba7/lib/set.rb#L401-L403
`Set`オブジェクト生成時に、内部的に生成した`@hash`に対してキー参照が行われ、O(1)で結果を取得できます。
:::

## 処理速度の検証
`Array#include?`と`Set#include?`の処理速度を比較して、どれくらい差があるのかを確認してみます。

### 検証用コード
```ruby
def benchmark_include(length)
  array = (1..length).to_a
  set = array.to_set

  Benchmark.bm do |x|
    x.report('Array:') { array.include?(length) }
    x.report('Set:') { set.include?(length) }
  end
end

[100, 100_0, 100_00, 100_000, 100_000_0].each do |length|
  pp "length: #{length}"
  benchmark_include(length)
end
```

### 検証結果
```ruby
"length: 100"
       user     system      total        real
Array:  0.000017   0.000000   0.000017 (  0.000003)
Set:  0.000003   0.000000   0.000003 (  0.000002)
"length: 1000"
       user     system      total        real
Array:  0.000006   0.000000   0.000006 (  0.000005)
Set:  0.000002   0.000000   0.000002 (  0.000002)
"length: 10000"
       user     system      total        real
Array:  0.000023   0.000012   0.000035 (  0.000035)
Set:  0.000002   0.000001   0.000003 (  0.000001)
"length: 100000"
       user     system      total        real
Array:  0.000232   0.000113   0.000345 (  0.000345)
Set:  0.000002   0.000001   0.000003 (  0.000002)
"length: 1000000"
       user     system      total        real
Array:  0.003023   0.000001   0.003024 (  0.003023)
Set:  0.000004   0.000001   0.000005 (  0.000003)
```
要素数が増えるにつれ、`Array#include?`の処理時間は比例して増加しますが、`Set#include?`は常に一定の時間で結果を返しています。

## 参考記事
https://qiita.com/an_sony/items/708c47d073ad709431d6
https://nekomaho.hatenablog.jp/entry/2019/05/28/083000
