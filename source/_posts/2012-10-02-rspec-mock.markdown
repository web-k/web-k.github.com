---
layout: post
title: "RSpecまとめ(2)～Mock(double/stub/mock)～"
date: 2012-10-02 13:59
comments: true
categories: [RSpec, Ruby]
---
[前回](/blog/2012/09/27/rspec_basic/)はRSpecの基本メソッドについてまとめました。今回はMockについてまとめます。

### テストダブルとは

テスト対象が依存しているモジュールやリソースの代役のこと。結合テストのような複雑な環境を事前に用意せずとも目的の機能をテスト可能となるように振る舞いをシミュレートする。

irb,pry等でMockを試したい時、
``` ruby
require 'rspec/mocks/standalone'
```
をrequireすると試せるのでやってみると良い

### double/stub/mock/stub_chain

おすすめの使い方で説明。他にも色々出来るとは思う

* **double** - ダミーオブジェクト作成
``` ruby
# ダミーオブジェクト作る。第1引数省略可能だが、なんかエラーとか出たら表示されるので入れておくと良い
book = double("book")
# book.title呼んだら"The RSpec Book"と返ってくるダミーオブジェクト
book = double("book", :title => "The RSpec Book")
```
* **stub** - ダミーメソッド作成
``` ruby
# 全部同じ。book.title呼んだら"The RSpec Book"と返ってくるダミーオブジェクト
book.stub(:title) { "The RSpec Book" }
book.stub(:title => "The RSpec Book")
book.stub(:title).and_return("The RSpec Book")
# doubleで作ったダミーオブジェクトでなくてもダミーメソッドを定義できる
String.stub(:test).and_return('test')
```
* **mock** - メソッドの振る舞いを評価するためのダミーオブジェクト作成
``` ruby
# logger.log('... Statement generated for Aslak ...') が呼ばれることを確認したい
it "logs a message on generate()" do
  # テストの為にcustomer.nameで'Aslak'を返しておきたいのでdouble/stub
  customer = double('customer')
  customer.stub(:name).and_return('Aslak')
  # 今回の評価対象なのでmock
  logger = mock('logger')
  statement = Statement.new(customer, logger)
  # logger.log('... Statement generated for Aslak ...') が呼ばれることを期待
  logger.should_receive(:log).with(/Statement generated for Aslak/)
  # テスト対象を実行して期待通りかテスト 
  statement.generate
end
```
* **stub_chain** - メソッドチェインの簡単な作成
``` ruby
Article.recent.published.authored_by(params[:author_id])
```
上記のメソッドチェインをまとめてstub化できる:
``` ruby
article = double()
Article.stub_chain(:recent, :published, :authored_by).and_return(article)
```

### stubとmockの違い

doubleもstubもmockもSpec::Mocks::Mockのインスタンスを作成します。テスト用途によって利用方法を振り分けます。

* **stub** - 呼び出しに対して決められた値を返します。結合すると環境構築が面倒だったり外部のサーバ状況に依っては失敗してしまうようなモジュール境界、ランダム値が返るオブジェクト等でダミーで返したい時に利用
* **mock** - どのように呼び出しされるか、そのダミーオブジェクトの振る舞い自体を評価したい場合に使用

### rspec-railsでのMock拡張

* **mock_model** - ダミーのmodelオブジェクトを作成します。ActiveRecordオブジェクトではなく、最低限のメソッドのみ対応しています。Controllerテストの様に正しくモデルを呼び出されているかチェックする時に利用します
``` ruby
# 評価対象はUser
user = mock_model(User)
# User.find_by_idが呼ばれた時にmockを返しておく
User.stub(find_by_id: user)
# user.saveが呼ばれることを期待
user.should_receive(:save).and_return(true)
# 期待通りに振る舞うかリクエスト
put '/users/1', login: 'zdenis'
```
``` ruby
# 保存前のダミーモデルオブジェクトとして振る舞う
new_user = mock_model(User).as_new_record
```
* **stub_model** - ActiveRecordオブジェクト本物をダミーとして作成します。Viewテストの様にModelを操作することが重要ではなく、値を参照して処理した結果、意図通りの生成結果となったかチェックする時に利用します
``` ruby
# @userに@user.loginが'zdennis'、@user.emailが'zdennis@example.com'のUserオブジェクトを登録
assign(:user, stub_model(User, login: 'zdennis', email: 'zdennis@example.com'))
render
renderd.should have_selector(...)
```

### 参考リンク

* 【書籍】[The RSpec Book](http://www.amazon.co.jp/RSpec-Book-Professional-Ruby-Series/dp/4798121932/ref=sr_1_1?ie=UTF8&qid=1349182365&sr=8-1) - RSpecの仕様書に留まらず、なぜBDDなのかが深く理解出来る良書になっています
* [rspec/rspec-mocks](https://github.com/rspec/rspec) - 本家
* [Chapter 14 Spec::Mocks - maeshimaの日記](http://d.hatena.ne.jp/maeshima/20100620/1277051360)
* [ricollab Web Tech Blog » Blog Archive » Mock と Stub について](http://blogs.ricollab.jp/webtech/2009/09/mock_and_stub/)
