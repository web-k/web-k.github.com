---
layout: post
title: "Ruby 2.0 キーワード引数"
date: 2013-01-17 15:00
comments: true
categories: ruby
author: aizawa1126 
---
Ruby 2.0.0の主な機能のうちのひとつ「キーワード引数」の挙動について。

キーワード引数はメソッドのそれぞれの引数にその意味を示すキーワードを付与し、キーワードを通じて引数を渡せる機能です。
```ruby
def my_method(str: "hoge", num: 1)
  [str, num]
end

# 実行結果
my_method # => ["hoge", 1]
my_method(str: "fuga") # => ["fuga", 1]
my_method(num: 2) # => ["hoge", 2]
my_method(strrr: "huga") # => ArgumentError: unknown keyword: strrr
```
実行時キーワード省略でデフォルト値が利用され、キーワード指定すると指定したものだけ値が書き換えられる。
引数全てを渡さなくてもよいというのも嬉しい。
定義されていないキーワードを指定すると当然ながらエラーとなる。
