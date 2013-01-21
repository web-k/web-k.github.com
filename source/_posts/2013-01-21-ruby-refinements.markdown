---
layout: post
title: "refinements"
date: 2013-01-21 14:06
comments: true
categories: ruby
author: aizawa1126 
---
ruby 2.0.0の主な機能のうちのひとつ「Refinements」の挙動について。

### Refinementsとは

既存のクラスの挙動を変更したり、拡張するときの影響範囲を名前空間で限定する機能です。
これまでも既存のクラスやメソッドに対する、動的な挙動の変更や追加は可能でしたが、以下のような問題もありました。

* プログラム全体に影響を与えてしまう
* 変更が見えにくい

モンキーパッチをモジュールとして分割してincludeするなどして、
明示的にするなどの対処法はとられてきましたが、
refinmentsは書き換えをより厳密で局所的なものにとどめるものになっています。

例としてStringクラスに局所的にメソッドを追加するコードを示します。
```ruby
module HogeExtension
  refine String do
    def to_camelcase
      split('_').map{|s| s.capitalize}.join('')
    end
  end
end

class RefineTest
  using HogeExtension

  def initialize
    p 'aaa_bbb_ccc'.to_camelcase
  end
end

RefineTest.new               # => "AaaBbbCcc"
p 'aaa_bbb_ccc'.to_camelcase # => NoMethodError
```
HogeExtensionモジュールでは、Stringクラスに対するto_calmelcaseメソッドの追加をrefineキーワードで行なっています。
この実装は、HogeExtensionモジュールをusingキーワードでとりこんだRefineTestクラスでは適用されるので、利用できます。
しかし、それ以外の部分でStringクラスのインスタンスに対しては、利用できないことがわかります。

### スコープ

refinementsは厳密なレキシカルスコープ(内部で定義した変数は外から見えないが、その逆は可能というスコープ)で動きます。
つまり、usingしたスコープと違うスコープに対しては、適用されません。
また、外側よりは内部の、前のよりは後の方が優先度が高く適用されます。
```ruby
module HogeExtension
  refine String do 
    def plus_string
      p self + ' hoge'
    end 
  end
end

module FugaExtension
  refine String do 
    def plus_string
      p self + ' fuga'
    end 
  end
end

class RefineTest
  using HogeExtension
  'outer1'.plus_string   # => "outer1 hoge"

  class Inner
    using FugaExtension
    'inner1'.plus_string # => "inner1 fuga"
  end

  'outer2'.plus_string   # => "outer2 hoge"

  using FugaExtension
  'outer3'.plus_string   # => "outer3 fuga"
end

# 拡張クラス内でも有効
class RefineTestEx < RefineTest
  'test'.plus_string     # => "test fuga"
end
```

### モジュールには適用できない

モジュールのメソッドにはrefinementsを適用できません。
```ruby
module EnumerableExt
  # エラー!
  refine Enumerable do
  end
end
```

### SelectorNamespcaceである

これはちょっとややこしいので、例を示します。
```ruby
class Hoge
  def say_hoge
    puts 'hoge'
  end

  def introduce
    print 'This is '
    say_hoge
  end
end

instance = Hoge.new
instance.say_hoge  # => 'hoge'
instance.introduce # => 'This is hoge'

module HogeExtension
  refine Hoge do
    def say_hoge
      puts 'fuga'
    end
  end
end

using HogeExtension
instance.say_hoge  # => 'fuga'
instance.introduce # => 'This is hoge'
```
26行目のintroduceに注目。
intstanceのintroduceが呼ぶのは元のAクラスのsay_hogeメソッドですので、'fuga'ではなく'hoge'の方が画面に出力されます。
これは、refinementsがレキシカルスコープで動くことの証拠です。
Aのintroduceメソッドは、それとは違うスコープで適用されたrefinementsの影響を受けません。
このような挙動をSelectorNamespcace と呼ぶそうです。
