---
layout: post
title: "Capybaraのブラウザエンジン"
date: 2012-11-07 16:37
comments: true
categories: [Ruby, capybara-webkit]
---
Capybaraは通常Rack:Testと呼ばれるブラウザエンジンを用い、仮想的に画面操作をしています。そのためJavascriptを考慮したテストはできません。
この場合、CapybaraのエンジンをJavascriptが動作する、capybara-webkitやSeleniumというブラウザエンジンに切り替えてテストを行います。
capybara_webkitを使うにはGemfileにcapybara-webkitを追記します。
``` ruby
group :test, :development do
  ...
  gem "capybara-webkit"
end
```
次にenv.rbにJavascriptドライバーを記述します。
``` ruby
Spork.prefork do
  ...
  Capybara.default_selector = :css
  Capybara.javascript_driver = :webkit
end
```
これでJacascripがテストで動作する環境ができました。Javascriptを動作させたいところにRSpecだと
``` ruby
it "hoge hoge", :js => true do
  ...
end
```
Cucumerだとシナリオに@javascriptタグを付加します。
``` ruby
@javascript
シナリオ: hoge hoge
```
ブラウザ別のテストを実施したいときはSeleniumドライバーを使います。
SeleniumドライバーはFirefox、InternetExplorer、GoogleChromeを実際に起動させてテストします。
SeleniumはCapybaraに同梱されているので、ブラウザエンジンを切り替えるだけで使用可能になります。
``` ruby
Capybara.default_driver = :selenium
```
デフォルトではFirefoxが起動するので、FirefoxではなくChromeを使用したいときには以下をCapybara.default_driverの前に記載します。
``` ruby
Capybara.register_driver :Selenium do |app|
  Capybara::Driver::Selenium.new(app, :browser => :chrome)
end
```

Seleniumドライバーは実際のブラウザを起動するのでほとんどのJavascriptをテストできますが、オーバーヘッドが大きくなります。
従って、必要なテストにのみSeleniumドライバーを利用するようにします。
RSpecのフィルタで切り替える方法を載せます。この方法は特定のサンプルのみブラウザの切り替えを行います。
フィルタ機能を設定するには、spec_helper.rbにSelenium用のフィルターを用意します。
``` ruby
config.before(:all, :selenium => true) do
  Capybara.current_driver = :selenium
end

config.after(:all, selenium => true) do
  Capybara.use_default_driver
end
```
あとはSeleniumを実行したいサンプルに:seleniumタグを追加します。
``` ruby
describe "hoge hoge", :selenium => true do
  ...
end
```
