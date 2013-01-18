---
layout: post
title: "Rubyのrequire/load/autoload/include/extendについて"
date: 2012-12-19 09:14
comments: true
author: ntth
categories: [Ruby]
---

### require/load/autoload/include/extend の違いについて
まず、ファイルをロードする require/load/autoload と ロードしない include/extend に分けられる。
違いについては、次で個別に説明した後に表にしてまとめる。

#### require

* Kernelモジュールのメソッド
* 同じファイルは複数回ロードされない
* Rubyライブラリをロードする
  * RubyライブラリはRubyスクリプト(\*.rb)と拡張ライブラリ(\*.so,\*.o,\*.dllなど)を指す
* ロードするファイルパスは、絶対パスでも相対パスでも可
* ロードするファイル名の拡張子は自動補完してくれるため、省略可(*.rb 優先)

{% codeblock lang:ruby %}
require 'ファイル名'
{% endcodeblock %}

#### load

* Kernelモジュールのメソッド
* 同じファイルを再ロードできる
* ロードするファイルパスは、絶対パスでも相対パスでも可
* ロードするファイル名の拡張子は省略できない
* 拡張ライブラリは読み込み対象ではない(できないかは不明)
* .txtのようなファイルでもロードできる

{% codeblock lang:ruby%}
load 'ファイル名'
{% endcodeblock %}

#### autoload

* Kernelモジュール(トップレベル)、Moduleクラス の2種類のメソッドがある
* autoloadと書いた段階では、ファイルはロードされない
  * 定数const_nameを参照したときに、はじめて第2引数であるファイルが require される
  * 定数const_nameには"::"演算子が使えないため、ネストする場合は2種類のメソッドを組み合わせて使用する
* いずれ廃止されそうなメソッド
  * [autoload will be dead - Ruby Forum](http://www.ruby-forum.com/topic/3036681)

{% codeblock lang:ruby %}
autoload(const_name, 'ファイル名')
{% endcodeblock %}

#### include

* Moduleクラスのメソッド
  * Moduleクラスはクラスとモジュールを表し、ClassクラスはModuleのサブクラスになる
* クラスやモジュールに他のモジュールをインクルード（Mix-in）する
  * 引数に指定できるのはモジュールのみ
  * 引数に複数のモジュール指定可
* includeしたクラスのインスタンスメソッドとして使用可能

{% codeblock lang:ruby %}include(module, ...){% endcodeblock %}

#### extend

* Objectクラスのメソッド
* オブジェクトの特異クラスに引数に指定したモジュールを取り込み、モジュールのメソッドを特異メソッドとして使えるようにする
  * 特異メソッドは特定のインスタンス固有のメソッドのこと
    * クラスを作った後にメソッドを追加すると特異メソッドになる
  * 特異クラスは特定のインスタンスのために用意されるクラスのこと
    * 特異クラスは特異メソッドをまとめて定義するときなどに使用される
* extendしたクラスのクラスメソッドとして使用可能(インスタンスメソッドではない)

{% codeblock lang:ruby %}extend(module, ...){% endcodeblock %}

extendは使用したことがなかったので、具体例を以下に示す。

{% codeblock lang:ruby extendの具体例 %}
module Test
  def test
    puts self + "Test"
  end
end
str = "Hoge"
str.extend(Test)
str.test
>> HogeTest
{% endcodeblock %}

### require/load/autoload の違い

　 | require  | load     | autoload
----------|----------|----------|----------
メソッドの定義元 | Kernelモジュール | Kernelモジュール | Kernelモジュール or Moduleクラス
複数回ロード | されない | される | されない(requireに従うので以下省略)
ロード時の拡張子省略 | 可 | 不可 |
ロード可能ライブラリ | Rubyスクリプト+拡張ライブラリ | Rubyスクリプト |

requireはライブラリのロードをしたいとき、loadは設定ファイルなど再読み込み可能にしたいとき、というように使い分けられる。
また、拡張子が.rbや.soなどではない.txt(中身はruby)ファイルをロードしたいときにも load を使用する。

### include/extend の違い

　 | include  | extend
----------|----------|----------
メソッドの定義元 | Moduleクラス | Objectクラス
振る舞い | includeしたクラスのインスタンスメソッドとして使用 | extendしたクラスのクラスメソッドとして使用

振る舞いの違いがわかりにくいので、以下に例を示す。

{% codeblock lang:ruby include/extendの例 %}
module Hoge
  def test
    puts "HogeTest"
  end
end

class IncludeTest
  include Hoge
end
class ExtendTest
  extend Hoge
end

IncludeTest.new.test #インスタンスメソッド
>> HogeTest
ExtendTest.test #クラスメソッド
>> HogeTest
{% endcodeblock %}

### 参考

* [オブジェクト指向スクリプト言語 Ruby リファレンスマニュアル](http://doc.ruby-lang.org/ja/1.9.3/doc/index.html)
* [Rubyリファレンス](http://ref.xaio.jp/ruby)
* [Rubyでrequireよりloadを使うべき場面は？ - QA@IT](http://qa.atmarkit.co.jp/q/2034)
* [requireとincludeとextendとmodule_function(1) - As Sloth As Possible](http://blog.livedoor.jp/faulist/archives/1220074.html)
* [Rubyで自作の外部モジュールを読み込む方法 - include と extend と module_function - (ﾟ∀ﾟ)o彡 sasata299's blog](http://blog.livedoor.jp/sasata299/archives/51268600.html)
