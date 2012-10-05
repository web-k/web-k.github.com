---
layout: post
title: "RSpecとCapybaraのマッチャ比較"
date: 2012-10-03 14:36
comments: true
categories: [RSpec, capybara]
---
Ruby on RailsでSpecを書く時に、RSpecのマッチャなのかCapybaraのマッチャなのか分からなくなったのでメモ

### RSpecのマッチャ

RSpecのマッチャには、「演算子マッチャ」と「ビルトインマッチャ」があり、「should」や「should_not」と組み合わせて使用します。
(shouldとshould_notはRSpecがObjectを拡張して作ったメソッド)

マッチャとして使用できる演算子は、「<」、「<=」、「==」、「===」、「=~」、「>」、「>=」の７種類。

使うときは以下のようにして記載します。
``` ruby
(1 + 2).should == 3
1.should < 2
"apple".should_not =~ /orange/
```
なお、「!=」や「!~」などの否定演算子は**サポートされていません**。「なんとかではないこと」を記載するときはshouldの代わりにshould_notを使います。

ビルトインマッチャは

* be
* be_a
* be_a_kind_of
* be_an_instance_of
* be_close
* be_within
* change
* eq
* eql
* equal
* exist
* expect
* have
* have_at_least
* have_at_most
* include
* match
* raise_error
* respond_to
* satisfy
* throw_symbol

って感じでたくさんあります。
「be_XXX」マッチャは「be_a_XXX」、「be_an_XXX」と記載しても同じ動作になります。
「have_XXX」マッチャは、RSpec の実行時には 「has_XXX?」メソッドの呼び出しとして解釈されます。これは、should と並べ たときの字面を (英語として) 自然な記述するための措置です。

使うときは以下のようにして記載します。
``` ruby
"string".should be_an_instance_of(String)
1.should be_a_kind_of(Numeric)
1.should_not be_an_instance_of(String)
```
また、マッチャを独自定義して使用することも可能です。
次のように書きます。（[公式ドキュメント](http://rspec.rubyforge.org/rspec/1.2.9/classes/Spec/Matchers.html)から引用）
``` ruby
Spec::Matchers.define :be_in_zone do |zone|
  match do |player|
    player.in_zone?(zone)
  end
end
```
Railsで使う場合は、spec/フォルダ直下にcustom_matchers.rbを作り、

custom_matchers.rb
``` ruby
module CustomMatcher
  def custom_matcher
    #マッチャの処理を記述 
  end
end
```
spec/spec_helper.rbを編集
``` ruby
require File.dirname(__FILE__) + "/custom_matchers"

RSpec.configure do |config|
  config.include(CustomMatcher)
```
これでマッチャとしてcustom_matcherが使えるようになります。

### Capybaraのマッチャ

Capybaraのマッチャは、オブジェクト対してメソッドの様に記載したり、RSpecのマッチャの様に記載することができます。
形式は「has_xxx?」の形をしているものがほとんど。

* assert_selector
* has_button? / has_no_button?
* has_checked_field? / has_no_checked_field?
* has_content? / has_no_content?
* has_css? / has_no_css?
* has_field? / has_no_field?
* has_link? / has_no_link?
* has_select? / has_no_select?
* has_selector? / has_no_selector?
* has_table? / has_no_table?
* has_text? / has_no_text?
* has_unchecked_field? / has_no_unchecked_field?
* has_xpath? / has_no_xpath?

って感じでこちらもたくさんあります。

使うときは以下のようにして記載します。
``` ruby
page.has_xpath?('//table/tr')
page.has_css?('table tr.hoge')
page.has_no_content?('hoge')
# page.has_XXX? は true/false を返すだけなので、テスト結果を得たいときは、
# assert page.has_xpath?('//table/tr')
# というようにassertを付けて記載します。
```
また、RSpecのマッチャの様に記載するときは「has_XXX?」を「have_XXX」に直して以下のようにします。
``` ruby
page.should have_xpath('//table/tr')
page.should have_css('table tr.hoge')
page.should have_no_content('hoge')
```
このとき、page.**should** have_no_XXX('value')をpage.**should_not** have_XXX('value')とは書かないほうがよいです。
理由はAjaxの待ち時間を考慮するためです。
Capybaraには非同期のJavascriptを扱う時に、まだページに存在していないされていないDOM要素を一定の待ち時間が経過したら再度探すという振る舞いがあります。
shouldを用いたときにその振る舞いは発揮されますが、should_notを用いたときはその振る舞いは発揮されず１回目の捜査で終了します。
待ち時間は次のように変更することができます。（デフォルトは2秒です。）
``` ruby
Capybara.default_wait_time = 3
```

### 参考リンク

* [Rubyist Magazine - スはスペックのス 【第 1 回】 RSpec の概要と、RSpec on Rails (モデル編)](http://jp.rubyist.net/magazine/?0021-Rspec)
* [rspecでカスタムマッチャの設定｜WEBデザイン Tips](http://blog.digital-squad.net/article/194513209.html)
* [Capybara の README 意訳 - おもしろWEBサービス開発日記](http://d.hatena.ne.jp/willnet/20110704/1309782442)
