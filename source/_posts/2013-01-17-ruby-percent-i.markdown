---
layout: post
title: "%i: シンボルの配列のリテラル"
date: 2013-01-17 15:59
comments: true
categories: ruby
author: aizawa1126
---
Ruby 2.0.0の主な機能のうちのひとつ「%i」の挙動について。

シンボルの配列がリテラルで書けるようになった。
```ruby
%i[hoge fuga] # => [:hoge, :fuga]
```
ちなみに文字列の配列のリテラルは以下のようにする。(これは以前からあった)
```ruby
%w[hoge fuga] # => ["hoge", "fuga"]
```
