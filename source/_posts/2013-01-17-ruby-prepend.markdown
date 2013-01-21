---
layout: post
title: "Module#prepend"
date: 2013-01-17 15:48
comments: true
categories: ruby
author: aizawa1126
---
Ruby 2.0.0の主な機能のうちのひとつ「Module#prepend」の挙動について。

prependは呼び出し元のクラス/モジュールの前にモジュールを置きます。
その呼ばれたモジュールの中で同じ名前を持つメソッドがあれば、それをラップします。
局所的なモンキーパッチを当てるような感じです。
```ruby
module Foo
  def foo
    p "before"
    super
    p "after"
  end
end

class Bar
  def foo
    p "bar"
  end
  prepend Foo
end

Bar.new.foo # "before", "bar", "after"
```
