---
layout: post
title: "Rails3 のroutes.rb"
date: 2012-12-18 23:14
comments: true
categories: 
author: aizawa1126
categories: [Rails]
---
Rails3になってルーティングの書き方がずいぶん変ったので使い方をメモ。

### 基本

Rails3 でのアプリケーション作り始めにて、
```
rails g scaffold article title:string
```
などと打つと、色々なファイルが生成され、その中にはroutes.rbも含まれている。routes.rbの覗いてみると、
``` ruby
resources :articles
```
という記載がある。
これはリソースCRUD操作を行うためのURLとアクションを自動で設定してくれる。
RailsでCRUDを行うために用意されている7つのアクション(index, new, create, show, edit, update, destroy)とURLとの紐付けをresourcesを使うことによって一度にすることができる。
なお、routes.rbにresources, resourceなどと書いていくわけだが、ルーティングの優先順位は上から順となっている。

現在設定されているルーティングを確認するには、rake routesコマンド用いる。
```
$rake routes
GET             /articles                 index        articles_path 
GET             /articles/new             new          new_article_path
POST            /articles                 create       articles_path
GET             /articles/:id             show         article_path(:id)
GET             /articles/:id/edit        edit         edit_article_path(:id)
PUT             /articles/:id             update       article_path(:id)
DELETE          /articles/:id             destroy      article_path(:id)
```

### id不要の場合

基本で記載したresourcesを用いた場合は、
```
GET             /articles/:id             show         article_path(:id)
```
のように、URLにidが付加されている。
しかし、ユーザのプロフィール設定画面などidが不要な場合もあるので、そのような場合はresourcesではなく、resourceを用いる。
```
resource :profile
```
7つのCRUDアクションの内、indexは生成されなくなる。
```
$rake routes
GET        /profile/new      new       new_profile_path
POST       /profile          create    profile_path
GET        /profile          show      profile_path
GET        /profile/edit     edit      edit_profile_path
PUT        /profile          update    profile_path
DELETE     /profile          destroy   profile_path
```

### ルーティングの制限

デフォルトのアクション7つのうち、不要なURLを生成したくない場合、:only 又は :except オプションを使用する。

:onlyオプションを用いると、指定したアクションのみ生成される。
``` ruby
resources :articles, :only => [:index, :new, :create, :show]
```
:exceptオプションを用いると、指定したアクションは生成されない。
``` ruby
resources :articles, :except => [:destroy]
```

### ルーティングの追加

ルーティングの追加方法はリソースのidがURLに付くかどうかで、2つの方法がある。リソースidが付く方(/articles/1/preview みたいなの)をメンバールーティング、付かない方(/articles/preview みたいなの)をコレクションルーティングと呼ぶ。

メンバールーティングの追加
``` ruby
resources :articles do
  member do
    get 'preview'
  end
end
# 又は
resources :articles do
  get 'preview', :on => :member
end
```
コレクションルーティングの追加
``` ruby
resources :articles do
  collection do
    get 'preview'
  end
end
# 又は
resources :articles do
  get 'preview', :on => :collection
end
```

### URLをピンポイントで指定

ログアウトのリンクなど、どのユーザでも変らないURLをピンポイントで指定するには次のようにmatchを用いる。
``` ruby
match "/authentication/logout" => "authentication#logout"
```

### メソッドの指定

``` ruby
get "/authentication/logout" => "authentication#logout"
# 又は :viaオプションを用いて
match "/authentication/logout" => "authentication#logout", :via => :get
```

### Named helperの設定

Named helperとはprofile_pathなどと書いているもののこと。
これを設定するには、:as オプションを使って、ルーティングのための名前を指定する。
```
match "/authentication/logout" => "authentication#logout", :as => :logout
```
これでlogout_pathやlogout_urlが使えるようになる。

### ネストしたURL

リソースが１対他の関係にある場合、URLをネストさせることができる。
例えば、1つの記事(article)にいくつかのコメント(comment)がある場合には次のように書くことができる。
``` ruby
resources :articles do
  resources :comments
end
```
便利だが、ネストしすぎると混乱を招きそうなので、1回くらいにしておくとよい。
2回、3回とネストしなければならない場合、設計を見直す機会かもしれない。
```
$rake routes
GET             /articles/:article_id/comments                 index        article_comments_path
GET             /articles/:article_id/comments/:id             show         article_comment_path(:id)
```

### トップページ(root)

デフォルトで用意されているpublic/index.htmlを消去してから、次のように記載する。
``` ruby
root :to => "articles#index"
```

### Non-Resourceful Routes

Non-Resourceful Routesとは、任意のURLをアクションにルーティングするためのサポート。
``` ruby
match ':controller(/:action(/:id))'
```
というように書く。これで、GET articles/edit/1 すると、パラメータは次のようになる。
``` ruby
{:controller => "articles", :action => "edit", :id => "1"}
```
URLに特定の文字列を含む場合は、
``` ruby
match ':controller/:action/:id/with_user/:user_id'
```
として、GET /articles/show/1/with_user/2すると
``` ruby
{:controller => "articles", :action => "show", :id => "1", :user_id => "2"}
```
となる。

参考サイト：[ruby/rails/RailsGuidesをゆっくり和訳してみたよ/Rails Routing from the Outside In - 株式会社ウサギィwiki](http://wiki.usagee.co.jp/ruby/rails/RailsGuides%E3%82%92%E3%82%86%E3%81%A3%E3%81%8F%E3%82%8A%E5%92%8C%E8%A8%B3%E3%81%97%E3%81%A6%E3%81%BF%E3%81%9F%E3%82%88/Rails%20Routing%20from%20the%20Outside%20In)
