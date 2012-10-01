---
layout: post
title: "RSpecまとめ～基本メソッド/Mock～"
date: 2012-09-27 01:53
comments: true
categories: [Ruby,RSpec]
---
RSpecで使う基本メソッド(describe/context/it/before/after/let)とMock(double/stub/mock)辺りをまとめてみる。

### 参考リンク

* [RSpec](http://kerryb.github.com/iprug-rspec-presentation/)
この記事はここのほぼ抜粋です。よくまとまっています
* [RSpec を使い始める人が読むべき N 個のドキュメント | Aiming 開発者ブログ](http://developer.aiming-inc.com/rails/rspec-references/)
参考リンクがいっぱいあります。上の記事もここで見つけた

### 基本メソッド

* describe/context - テスト名。テスト対象自身のオブジェクト(subject代わり)でもOK
* before - 事前条件。example(it)の前に実行される。before :each はexample毎、before :all はdescribe毎
* after - 事後処理
* subject - 評価対象。shouldの前のオブジェクトを指定することでit内のオブジェクトを省略できる
* it - テスト仕様。マッチャーを使って評価する

{% codeblock lang:ruby %}
describe Controller do
  # 一番上のdescribeはテスト対象にしておくといい。subjectにもなる

  context "handling AccessDenied exceptions" do
    # contextはdescribeのalias。テスト対象にgetのときとか事前条件変えるときはこっちの方が読みやすいかも

    before { get :index } # 1行になるべく書く
      # デフォルトはbefore :each
      # 事前条件はitにも書けるけど、なるべく分けるように

    subject { response } # itのテスト対象。shouldの前を省ける

    it { should redirect_to("/401.html") } # itの後に概要書けるけど、マッチャーだけで意味通るなら省略する

  end
end
{% endcodeblock %}

* it(context, tags) - タグが付けられる

{% codeblock lang:sh %}
rspec --tag ruby:1.8
{% endcodeblock %}

{% codeblock lang:ruby %}
describe "Something" do
  it "behaves one way in Ruby 1.8", :ruby => "1.8" do
    ...
  end
 
  it "behaves another way in Ruby 1.9", :ruby => "1.9" do
    ...
  end
end
{% endcodeblock %}

* its - subjectのメソッドが呼べる

{% codeblock lang:ruby %}
describe [1, 2, 3, 3] do
  its(:size) { should == 4 }
  its("uniq.size") { should == 3 }
end
{% endcodeblock %}
