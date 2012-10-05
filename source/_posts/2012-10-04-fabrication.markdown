---
layout: post
title: "Fabrication"
date: 2012-10-04 12:02
comments: true
categories: [Ruby,Fabrication]
---

ここでは、[Fabrication](http://www.fabricationgem.org/) のサイトを日本語に直して、自己解釈して補完しながら説明していきます。

## Fabricationとは

これはオブジェクト生成ライブラリで、
オブジェクトの概略だけを定義し、素早くオブジェクトを使うことができるものである。
サポートしているオブジェクトタイプは以下のものなどがある。

* ActiveRecord Models
* Mongoid Documents
* Sequel Models
* DataMapper Resources
* ・・・

## 設定

Gemfile に Fabrication を記載し、bundle install すれば使える
{% codeblock lang:ruby %}
gem 'fabrication'
{% endcodeblock %}
デフォルトでは以下にFabrication関連のソースを置くと、自動ロードされる。
{% codeblock %}
spec/fabricators/**/*fabricator.rb
test/fabricators/**/*fabricator.rb
{% endcodeblock %}
設定を変更したい場合は以下のように、Fabrication.configureで設定変更できる。
{% codeblock lang:ruby %}
Fabrication.configure do |config|
  config.fabricator_path = 'data/fabricators' #Fabrication関連の定義を置くパス
  config.path_prefix = Rails.root #ファイルシステムへの許可範囲
end
{% endcodeblock %}

## 引数

{% codeblock lang:ruby %}
class Person; end
Fabricator(:person) #引数がFabricatiorオブジェクトになる。クラス名のシンボルである必要がある
{% endcodeblock %}

{% codeblock lang:ruby %}
Fabricator(:adult, from: :person) #from: :symbolized_class_nameのクラス名を変えて:adultというFabricatiorオブジェクトが定義できる
{% endcodeblock %}

## 属性

Fabricator ブロックには変数が必要ではないが、1つ提供される。属性リスト作成時に、宣言もされる。

{% codeblock lang:ruby %}
Fabricator(:person) do
  name 'Greg Graffin'
  profession 'Professor/Musician'
end
{% endcodeblock %}

属性には変数を渡すことができる
{% codeblock lang:ruby %}
Fabricator(:person) do
  name { Faker::Name.name }
  profession { %w(Butcher Baker Candlestick\ Maker).sample }
end
{% endcodeblock %}

属性は処理順に宣言され、上記フィールドのブロック変数を用いることができる。
{% codeblock lang:ruby %}
Fabricator(:person) do
  name { Faker::Name.name }
  email { |attrs| "#{attrs[:name].parameterize}@example.com" }
end
{% endcodeblock %}

## 予約語

予約語名をブロック変数と一緒に使うことで属性として参照できる
{% codeblock lang:ruby %}
Fabricator(:person) do |f|
  f.alias 'James Bond'
end
{% endcodeblock %}

## 関連

他のFabricatorに関連付ける場合は、属性名を書くだけでいい。
これで、「**belongs_to**」の関連を表現できる。

{% codeblock lang:ruby %}
Fabricator(:person) do
  vehicle
end
↑↓等価
Fabricator(:person) do
  vehicle { Fabricate(:vehicle) }
end
{% endcodeblock %}

{% codeblock lang:ruby %}
Fabricator(:person) do
  ride(fabricator: :vehicle)
end
↑↓等価
Fabricator(:person) do
  ride { Fabricate(:vehicle) }
end
{% endcodeblock %}

countパラメータを使うことで、配列オブジェクトを生成できる。

{% codeblock lang:ruby %}
Fabricator(:person) do
  open_souce_projects(count: 5)
  children(count: 3) { |attrs, i| Fabricate(:person, name: "Kid #{i}") }
end
{% endcodeblock %}

## 継承

他の Fabricators から属性を継承する場合は、「**:from**」 を使う
{% codeblock lang:ruby %}
Fabricator(:llc, from: :company) do #クラスの属性とその値を全て継承する
  type "LLC"
end

Fabricator(:llc, class_name: :company) do # class_name: でクラスの属性のみを継承する
  type "LLC"
end
{% endcodeblock %}

## 初期化

オブジェクトの初期化を通常の方法でしてほしくないときは、
**initialize_with**を以下のようにオーバーライドすればよい。
{% codeblock lang:ruby %}
Fabricator(:car) do
  initialize_with { Manufacturer.produce(:new_car) }
  color 'red'
end
{% endcodeblock %}

## コールバック

Fabricationのビルドにフックするには、**after_build**、**after_create** を使う。
{% codeblock lang:ruby %}
Fabricator(:place) do
  after_build { |place| place.geolocate! } #ビルド後=保存する前
  after_create { |place| Fabricate(:restaurant, place: place) } #保存した後
end
{% endcodeblock %}
オブジェクトに引数を与えたときのコンストラクタでコールバックするときは、**on_init**を使う。
{% codeblock lang:ruby %}
Fabricator(:location) do
  on_init { init_with(30.284167, -81.396111) }
end
{% endcodeblock %}
コールバックはスタックになっているので、並列にFabricatorを宣言できるし、継承しても大丈夫。

## エイリアス

Fabricatior呼び出し時に**:aliases**オプションをつけるとエイリアスが付けれる。
{% codeblock lang:ruby %}
Fabricator(:thingy, aliases: [:widget, :wocket]) #Fabricateを :thingy,:widget, :wocketどれでも呼び出せる
{% endcodeblock %}

## 一時属性

Fabricator内で一時属性を変数として持てるが、クラス生成時にはセットされない。
一時属性は、クラスが生成されるまでの間は普通の属性と同じように扱えるが、生成時に取り除かれる。
{% codeblock lang:ruby %}
Fabricator(:city) do
  transient :asian
  name { |attrs| attrs[:asian] ? "Tokyo" : "Stockholm" }
end
Fabricate(:city, asian: true)
# => <City name: 'Tokyo'>

Fabricator(:the_count) do
  transient :one, :two, :three #複数定義可
end
{% endcodeblock %}

## リロード

Fabricationがロードされた状態にリセットする
{% codeblock lang:ruby %}
Fabrication.clear_definitions
{% endcodeblock %}

## 基本

Fabricateオブジェクトを作成する簡単な方法は、クラス名を渡すだけでいい。
{% codeblock lang:ruby %}
Fabricate(:person)
{% endcodeblock %}
これで、PersonのインスタンスがFabricatorとして定義される。
Fabricator作成時に、引数としてハッシュを渡せば、属性の追加や、上書きができる。
{% codeblock lang:ruby %}
Fabricate(:person, first_name: "Corbin", last_name: "Dallas")
{% endcodeblock %}

## Fabricating With Blocks

Fabricateのブロックの引数にハッシュ値を渡せば、オブジェクト生成時に定義され利用できる。
{% codeblock lang:ruby %}
Fabricate(:person, name: "Franky Four Fingers") do
  addiction "Gambling"
  fingers(count: 9)
end
{% endcodeblock %}

## ビルド

データベースにオブジェクトを持続させたくないときは Fabricate.build を使う。
{% codeblock lang:ruby %}
Fabricate.build(:person)
{% endcodeblock %}
下記のように、Fabricate.build内でFabricateが呼ばれていてもオブジェクトは持続せず、buildのときの動作と同じになる。
{% codeblock lang:ruby %}
Fabricate.build(:person) do
  cars { 2.times { Fabricate(:car) } }
end
{% endcodeblock %}

## 属性のハッシュ

オブジェクトを生成せずに、属性だけを生成してハッシュで返したい場合は以下のようにする。
{% codeblock lang:ruby %}
Fabricate.attributes_for(:company)
{% endcodeblock %}

## Sequences

Sequencesはそのプロセスにおいての、ユニークな連続した数値が得られる。
Sequencesは指定がなければ、0からはじまる。

{% codeblock lang:ruby %}
Fabricate.sequence #実行するたびにインクリメントされていく
# => 0 #1回目
# => 1 #2回目
# => 2 #3回目

Fabricate.sequence(:name) #引数を渡せば独自の数値でインクリメントされる
# => 0
# => 1
# => 2

Fabricate.sequence(:number, 99) #第2引数の数値は開始の数値
# => 99
# => 100
# => 101

Fabricate.sequence(:name) { |i| "Name #{i}" } #ブロックで渡してもインクリメントされる
# => "Name 0"
# => "Name 1"
# => "Name 2"

Fabricate(:person) do #例
  ssn { sequence(:ssn, 111111111) }
  email { sequence(:email) { |i| "user#{i}@example.com" } }
end
# => <Person ssn: 111111111, email: "user0@example.com">
# => <Person ssn: 111111112, email: "user1@example.com">
# => <Person ssn: 111111113, email: "user2@example.com">
{% endcodeblock %}

# Rails 3

Rails 3でFabricatorsをモデル生成時に一緒に生成したい場合は、config/application.rb に設定を書く。
{% codeblock lang:ruby %}
# rspecの場合
config.generators do |g|
  g.test_framework      :rspec, fixture: true
  g.fixture_replacement :fabrication
end
# test/unitの場合
config.generators do |g|
  g.test_framework      :test_unit, fixture_replacement: :fabrication
  g.fixture_replacement :fabrication, dir: "test/fabricators"
end
# minitestの場合
config.generators do |g|
  g.test_framework      :mini_test, fixture_replacement: :fabrication
  g.fixture_replacement :fabrication, dir: "test/fabricators"
end
{% endcodeblock %}
上記設定後、下記コマンドでFabricationのファイルができる。

{% codeblock lang:ruby %}
$ rails generate model widget #コマンド
spec/fabricators/widget_fabricator.rb
 Fabricator(:widget) do #中身
 end
{% endcodeblock %}

### 参考リンク

* [Fabrication:本家](http://www.fabricationgem.org/)
* [Fabricationを使ってみた - のどをRubyでいっぱいにして](http://d.hatena.ne.jp/hibariya/20101010/1286713523)
* [あーありがち - 素の Ruby 環境で Fabrication](http://aligach.net/diary/20101220.html)
* [Fabrication:SlideShare](http://www.slideshare.net/KenichiTachibana/fabrication-10499288)


