---
layout: post
title: "RSpecで例外のテストをする"
date: 2012-11-07 23:14
comments: true
categories: RSpec
---
RSpecで例外のテストするにはlambdaを使用する。
例えば、adminというroleを持つUserのインスタンスに対し、destroyというインスタンスメソッドを実行すると、CannotDestroyAdminUserという例外が出るテストをする。その場合次のように書く。
```ruby
it "hoge hoge" do
  @user.roles=(['admin'])
  @user.save
  lambda{@user.destroy}.should raise_error(CannotDestroyAdminUser)
end
```
