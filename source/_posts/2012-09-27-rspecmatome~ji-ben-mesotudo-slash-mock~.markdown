---
layout: post
title: "RSpecまとめ～基本メソッド/Mock～"
date: 2012-09-27 01:53
comments: true
categories: RSpec
---
RSpecで使う基本メソッド(describe/context/it/before/after/let)とMock(double/stub/mock)辺りをまとめてみる。

### 参考リンク

* [RSpec](http://kerryb.github.com/iprug-rspec-presentation/)
この記事はここのほぼ抜粋です。よくまとまっています
* [RSpec を使い始める人が読むべき N 個のドキュメント | Aiming 開発者ブログ](http://developer.aiming-inc.com/rails/rspec-references/)
参考リンクがいっぱいあります。上の記事もここで見つけた

### 基本メソッド

{% codeblock lang:ruby %}
describe Controller do
  # 一番上のdescribeはテスト対象にしておくといい。subjectにもなる

  context "handling AccessDenied exceptions" do
    # contextはdescribeのalias。テスト対象にgetのときとか事前条件変えるときはこっちの方が読みやすいかも

    before do
      # デフォルトはbefore :each
      # 事前条件はitにも書けるけど、なるべく分けるように
      get :index
    end

    it "redirects to the /401.html page" do
      response.should redirect_to("/401.html")
    end


  end
end
{% endcodeblock %}


