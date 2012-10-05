---
layout: post
title: "FabricationとCucumberの連携"
date: 2012-10-05 10:13
comments: true
categories: [Ruby,Fabrication,Cucumber]
---

前回Fabricationの続きで、Cucumberと連携する方法についてここに記載します。

## Cucumber

### インストール

step_definitionsフォルダに便利な cucumber_steps を生成してくれるツールがgemの中にパッケージ化されている。
Gemfileのdevelopment環境にcucumber系を含めている必要がある。
{% codeblock lang:ruby %}
$ rails generate fabrication:cucumber_steps
# => create  features/step_definitions/fabrication_steps.rb
{% endcodeblock %}

### Step Definitions

WidgetモデルのFabricatorが定義されていれば、下記のように書くだけでFabricateできる。
{% codeblock lang:cucumber %}
Given 1 widget
{% endcodeblock %}
Widgetモデルの属性を指定してFabricateすることもできる。
{% codeblock lang:cucumber %}
Given the following widget:
  | name      | widget_1 |
  | color     | red      |
  | adjective | awesome  |
{% endcodeblock %}
複数も可
{% codeblock lang:cucumber %}
Given 10 widgets
{% endcodeblock %}
属性付きで複数Fabricateする。
{% codeblock lang:cucumber %}
Given the following widgets:
  | name     | color | adjective |
  | widget_1 | red   | awesome   |
  | widget_2 | blue  | fantastic |
  ...
{% endcodeblock %}
既にFabricateされた"widget"に"wockets"を所属させる。
{% codeblock lang:cucumber %}
And that widget has 10 wockets
{% endcodeblock %}
既にFabricateされた"widget"に"wockets"を属性を与えて所属させる。
{% codeblock lang:cucumber %}
And that widget has the following wocket
  | title    | Amazing |
  | category | fancy   |
{% endcodeblock %}
既にFabricateされた"widget"と"wockets"を関連付ける。
{% codeblock lang:cucumber %}
And that wocket belongs to that widget
{% endcodeblock %}
データベースにいくつのオブジェクトが保持されているか検証する。
{% codeblock lang:cucumber %}
Then I should see 1 widget in the database
{% endcodeblock %}
オブジェクトの中身も検証できる。
{% codeblock lang:cucumber%}
Then I should see the following widget in the database
  | name  | Sprocket |
  | gears | 4        |
  | color | green    |
{% endcodeblock %}

### Transforms

cucumberのステップでテーブルを変換できる。縦横のテーブルでカラムの値を再配置できる。
spec/fabricatorsフォルダにおいておけば、何とでも設定しておける。

例として、全てのフィールドの"company"に変換の定義をする。lambda には返り値の属性をセットしたい文字列を置く。
その結果、"company"のインスタンスオブジェクトが生成される。
{% codeblock lang:ruby%}
Fabrication::Transform.define(:company, lambda{ |company_name| Company.where(name: company_name).first })
{% endcodeblock %}
{% codeblock lang:cucumber%}
Scenario: a single object with transform to apply
  Given the following company:
    | name | Widgets Inc |
  Given the following division:
    | name    | Southwest   |
    | company | Widgets Inc |
  Then that division should reference that company

Scenario: multiple objects with transform to apply
  Given the following company:
    | name | Widgets Inc |
  Given the following divisions:
    | name      | company     |
    | Southwest | Widgets Inc |
    | North     | Widgets Inc |
  Then they should reference that company
{% endcodeblock %}
divisions を生成したときに、lambdaによって"company"オブジェクトに渡されている。

特定のモデルのスコープにだけ適用したい場合は、**only_for**を使う。
{% codeblock lang:ruby%}
Fabrication::Transform.only_for(:division, :company, lambda { |company_name| Company.where(name: company_name).first })
{% endcodeblock %}
