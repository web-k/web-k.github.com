---
layout: post
title: "Railsのbefore_filterとメソッド返り値"
date: 2012-10-09 16:54
comments: true
categories: 
---
before_filterで実行したメソッドがfalseをreturnしたらどうなるか、気になったのでメモ。

確認した環境はRuby 1.9.3p194, Rails 3.2.8。
``` ruby
class ApplicationController < ActionController::Base

  before_filter :false_filter
#  before_filter :redirect_filter
  before_filter :true_filter

  def false_filter
    return false
  end

  def redirect_filter
    redirect_to root_path
  end

  def true_filter
    return true
  end

```
true/falseだからといって特に何も起こらなかった。falseでもそのまま次のbefore_filterが実行されたり、filter後に控えているコントローラのメソッドが実行された。

before_filterやafter_filterはメソッドを優先して実行するかどうかを決めているだけであり、メソッドの返り値を受け取ってどうこうするというモノではないようだ。

ただし、before_filterで呼んだメソッドの中にrenderやredirect_to、raiseなどがあると、その後に控えている他のbefore_filterや以降のコントローラのメソッドは実行されない。
