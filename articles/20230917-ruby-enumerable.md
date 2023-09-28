---
title: "【Ruby】Enumerableとはなんぞや"
emoji: "💎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ruby"]
published: true
published_at: 2023-09-17 08:00
---

## Enumerable とは

公式リファレンスによると「繰り返しを行なうクラスのための Mix-in」とのこと。
Mix-in は「モジュールをクラスにインクルードして機能を追加すること」を表すので、どうやら Array や Hash 等の繰り返し処理が行えるクラスにインクルードされているモジュールのようです。
読み方は"イニュメラブル"でいいのかな？（調べてもそれっぽいのが見つからなかった…）

インクルードするクラスには each が定義されている必要があるみたいです。
裏を返せば each さえ定義できていれば map や sort などの、処理に繰り返しが必要になる便利なメソッドが、インクルードするだけで利用できるということですね。

## おまけ

実際に ruby の内部実装を見てみたところ、しっかりインクルード(=`rb_include_module`)されていました。

https://github.com/ruby/ruby/blob/8835ca23c138b2fa5e883acd6b368fdc25d7ce23/array.c#L8603-L8607

https://github.com/ruby/ruby/blob/8835ca23c138b2fa5e883acd6b368fdc25d7ce23/hash.c#L7047-L7056
