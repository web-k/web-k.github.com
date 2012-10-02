---
layout: post
title: "RSpecまとめ(1)～基本メソッド～"
date: 2012-09-27 01:53
comments: true
categories: [Ruby,RSpec]
---
RSpecで使う基本メソッド(describe/context/it/its/before/after/subject/let/shared_examples_for)をまとめてみる。

### 参考リンク

* [RSpec](http://kerryb.github.com/iprug-rspec-presentation/) - 本記事はここのほぼ抜粋です。よくまとまっています
* [RSpec を使い始める人が読むべき N 個のドキュメント | Aiming 開発者ブログ](http://developer.aiming-inc.com/rails/rspec-references/) - 参考リンクがいっぱいあります。上の記事もここで見つけた

### 基本メソッド

* **describe/context** - テスト名。テスト対象自身のオブジェクト(subject代わり)でもOK
* **before** - 事前条件。example(it)の前に実行される。*before :each* はexample毎、*before :all* はdescribe毎に呼ばれる
* **after** - 事後処理。以後のテストに影響が出ないように後始末が必要な時に記述する
* **subject** - 評価対象。shouldの前のオブジェクトを指定することでit内のオブジェクトを省略できる
* **it** - テスト仕様。マッチャーを使って評価する

``` ruby
# 一番上のdescribeはテスト対象にしておくといい。subjectにもなる
describe Controller do
  # contextはdescribeのalias。テスト対象にgetのときとか事前条件変えるときはこっちの方が読みやすいかも
  context "handling AccessDenied exceptions" do
    # デフォルトはbefore :each
    # 事前条件はitにも書けるけど、なるべく分けるように
    before { get :index } # 1行になるべく書く

    # itのテスト対象。shouldの前を省ける
    subject { response }

    # itの後に概要書けるけど、マッチャーだけで意味通るなら省略する
    it { should redirect_to("/401.html") }
  end
end
```

* **it(context, tags)** - タグが付けられる

<pre>$ rspec --tag ruby:1.8</pre>

``` ruby
describe "Something" do
  # ruby:1.8
  it "behaves one way in Ruby 1.8", :ruby => "1.8" do
    ...
  end

  # ruby:1.9
  it "behaves another way in Ruby 1.9", :ruby => "1.9" do
    ...
  end
end
```

* **its(method)** - subjectのメソッドが呼べる

``` ruby
describe [1, 2, 3, 3] do
  # [1,2,3,3].size.should == 4
  its(:size) { should == 4 }

  # [1,2,3,3].uniq.size.should == 3
  its("uniq.size") { should == 3 }
end
```

* **let** - example内で同じオブジェクトの使い回し

``` ruby
describe BowlingGame do
  # example毎に呼ばれる。でも遅延評価。使わなかったら呼ばれない
  let(:game) { BowlingGame.new }
 
  it "scores all gutters with 0" do
    20.times { game.roll(0) }
    game.score.should == 0
  end
 
  it "scores all 1s with 20" do
    20.times { game.roll(1) }
    game.score.should == 20
  end
end
```

* **shared_examples_for** - exampleの共通化

``` ruby
shared_examples_for "a single-element array" do
  # letやbeforeやafterも書ける。要はなんでも共通化
  let(:xxx) { Obj.new }
  before { puts 'before' }
  after { puts 'after' }

  # subjectやletを渡せる
  it { should_not be_empty }
  it { should have(1).element }
end
 
describe ["foo"] do
  it_behaves_like "a single-element array"
end
 
describe [42] do
  it_behaves_like "a single-element array"
end
```

* **shared_context** - 事前条件の共通化

``` ruby
shared_context '要素がふたつpushされている' do
  let(:latest_value) { 'value2' }
  before do
    @stack = Stack.new
    @stack.push 'value1'
    @stack.push latest_value
  end
end

describe Stack do
  describe '#size' do
    include_context '要素がふたつpushされている'
    subject { @stack.size }
    it { should eq 2 }
  end

  describe '#pop' do
    include_context '要素がふたつpushされている'
    subject { @stack.pop }
    it { should eq latest_value }
  end
end
```

次回はMockについてまとめてみます。
