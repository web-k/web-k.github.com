---
layout: post
title: "Assets Pipeline"
date: 2013-01-09 00:51
comments: true
author: toshi3221
categories: [Rails, assets]
---
## この記事は

基本は[Ruby on Rails Guides: Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html)の訳ですが簡略化や自分の解釈で意訳した部分が多々あります。気になる点あったらコメントください。

## Assets Pipelineとは

Asset(アセット)とは、訳すと「資産」のこと。Assets Pipelineは画像やJavaScript、CSSを高速でリクエストを捌けるようにしたRails 3.1より標準搭載された仕組みです。

## Assets Pipelineで出来ること

* Fingerprinting
  * コンテンツベースのファイル名に更新することによるキャッシュ支援
* Precompileを利用した高レベル言語でのコーディング
  * CSSに対してSass/SCSS/LESS、JavaScriptに対してCoffeeScript等の中間言語が使用可能
  * ERBも使用可能
* Assetの連結
  * 複数のJavascriptやCSSをまとめてリクエスト数を減らす
* Assetの最小化と圧縮
  * CSSは空白削除、JavaScriptはif-elseを三項演算子化など、サイズ縮小のために最適化

Assets Pipelineはデフォルトで有効です。無効にしたければconfig/application.rbにて

```
config.assets.enabled = false
```

を設定してください。また、アプリケーション作成時であれば

```
rails new appname --skip-sprockets
```

と--skip-sporocketsオプションを付ける事でも無効にできます。ただし、何らかの理由が無ければ、デフォルトの状態で有効にしておくと高速化の面でも有利ですのでそのままにしておくべきでしょう。

## Assets Pipelineの使い方

app/assetsディレクトリの中にAssetを配置します。今まではpublicに全て配置していて、利用は可能ですが、全て静的ファイルとしてのみ扱われます。

production環境のデフォルトでは事前にプリコンパイルを行い、public/assetsに配置して動作時にapp/assetsから直接出力せずパフォーマンスに配慮したデプロイを行います。

Assetの配置場所は

* app/assets
* lib/assets
* vendor/assets

の三箇所があります。appはアプリケーション固有のもの、libは複数のアプリケーションでも利用できるもの、vendorはサードベンダーのJSプラグインやCSSフレームワークといったように使い分けると良いでしょう。


### Asset Pathの検索

マニフェストやヘルパーよりファイルを参照すると、デフォルトではimages、javascripts、stylesheetsのサブディレクトリ内からファイルを探します。例えば、

```
app/assets/javascripts/home.js
lib/assets/javascripts/moovinator.js
vendor/assets/javascripts/slider.js
```

をJSマニフェストから参照するとすると

```
//= require home
//= require moovinator
//= require slider
```

ヘルパからだと

```
<%= javascript_include_tag "home" %>
<%= javascript_include_tag "moovinator" %>
<%= javascript_include_tag "slider" %>
```

にて参照できます。

indexファイルはフォルダ名指定でlib/assets/library_name/index.jsを

```
//= require library_name
```

と参照可能です。ちなみにRails.application.config.assets.pathsを見るとAsset検索Pathが分かります。また、以下のようにconfig/application.rbにパスを追加できます

```
config.assets.paths << Rails.root.join("app", "assets", "flash")
```

重要なことは、検索したいファイルはプリコンパイル対象となっていることです。そうしないとproduction環境では利用できなくなってしまいます。

## Fingerprintingとは

Fingerprintingとはファイル内容も加味されたファイルネーミングの仕組みです。ファイルの中身が変更されるとファイル名も更新されます。

これにより、CDNであったり、複数の別のサーバー上に配置したとしても、ファイル名によってコンテンツ内容が一意である事が分かり、キャッシュ最適化を行う事が可能になります。

Assets Pipelineではファイル名にMD5ハッシュが挿入されます。global.cssが元ファイルとすると、global-908e25f4bf641868d8683022a5b62f54.cssのようなファイル名となります。

Assets Pipeline導入前のRailsは時間ベースの文字列クエリを付加していましたが、以下のデメリットがありました：

* クエリだと確実にキャッシュする事を保証出来ない
  * CDN等でリクエストがキャッシュされないケースがあった
* 複数のサーバに静的コンテンツを配置するとファイル名が異なる
  * ファイル時間が同一ファイルでも異なる事があるために無駄に読み込みがかかるケースがあった
* 同一のファイルでも再度デプロイしたらキャッシュが利用されないことがある
  * 更新日時が変更されたら再度読み込みを強いることになる

これらのデメリットをファイル内容に基づくファイル名の付加によって解消されました。

Fingerprintingはproduction環境で有効、その他の環境は無効にデフォルトはなっています。替えたい場合はconfig.assets.digest設定を変更して下さい。

### Assetへのリンク

Assets Pipelineでは通常のリンクヘルパを利用すればOKです

```
<%= stylesheet_link_tag "application" %>
<%= javascript_include_tag "application" %>
<%= image_tag "rails.png" %>
```

accetsパスへの検索、production環境であればMD5をファイル名に付加更新もここで行われます

```
rails-af27b6a414e6da00003503148be9b409.png
```

#### CSS/JavaScript/ERB

Assets PipelineはERBにて評価可能です。application.css.erbのようにerbの拡張子を付加すれば次のように利用できます。

```
.class { background-image: url(<%= asset_path 'image.png' %>) }
```

Base64形式として埋め込みヘルパもあります

```
#logo { background: url(<%= asset_data_uri 'logo.png' %>) }
```

application.js.cooffee.erbのようにCoffeeScriptにも利用できます

```
$('#logo').attr src: "<%= asset_path('logo.png') %>"
```

### アセットの依存関係と連結～マニフェストファイルとディレクティブ

どのアセットを利用するかを決定するのにマニフェストファイルを利用します。CSS/JavaScriptにコメントにてディレクティブを記述します。また、Rails.application.config.assets.compressがtrueであれば連結します。連結することに単一ファイルになるのでリクエスト時間を減らせます。例えばapp/assets/javascripts/application.jsに以下のように記述します

```
// ...
//= require jquery
//= require jquery_ujs
//= require_tree .
```

//=で行を始めてディレクティブを記載します。jquery.jsとjquery_ujs.jsはjquery-rails gemが提供しています。

require_treeは再帰検索を行います。対象パスのサブディレクトリも検索してすべてのファイルを出力します。サブディレクトリを検索しないで特定のディレクトリを指定したい場合はrequire_directoryを利用して下さい。

requireは上から順に処理されますが、require_treeは何処に記載しても順序は関係無いです。連結した場合でも、特定のJavascriptが一番最初に来るように保証したいのであれば、requireも一番最初に記述してください

CSSの例は以下のようになります

```
/* ...
*= require_self
*= require_tree .
*/
```

requre_selfはこれが呼び出された順序で自分のCSSを読み込むようになります。require_treeを使うとJSと同様に現在のディレクトリのすべてのスタイルシートを読み込みます。

複数のSassファイルを利用する時はSassの@importルールを使用スべきです。ディレクティブで記載してしまうと全Sassファイルはそれぞれのスコープ下に存在することになって変数やmixinが定義されたドキュメント内でしか利用できなくなります。

### Preprocessing

Assetの拡張子で前処理が決まります。projects.js.coffeeとすればCoffeeScriptが、projects.css.scssとすればSCSSが処理されます。

複数の拡張子を付加すれば前処理の多重化も可能です。projects.css.scss.erbとすれば、拡張子は右側からERB、SCSSと処理され、最終的にCSSとして応答します。

この拡張子の処理順序は重要で、projects.js.erb.coffeeと記載してしまうと、ERB記述のままCoffeeScriptインタプリタが処理されてしまいエラーとなってしまいます。projects.js.coffee.erbの順で記載してください。

### JavaScriptの圧縮

JavaScriptは:closure :uglifier :yuiの3種類が選べます。closure-compiler・uglifier・yui-compressorの各gemが必要です。Gemfileにはデフォルトでuglifierがあります。ホワイトスペースを削除し、if-else文を三項演算子に変更するなどの最適化をします。

```
config.assets.js_compressor = :uglifier
```

JS圧縮を有効にするには以下の設定が必要です

```
config.assets.compress = true
```

### Development環境

development環境では連結されずに別々のファイルとなります

```
//= require core
//= require projects
//= require tickets
```

の記載では

```
<script src="/assets/core.js?body=1" type="text/javascript"></script>
<script src="/assets/projects.js?body=1" type="text/javascript"></script>
<script src="/assets/tickets.js?body=1" type="text/javascript"></script>
```

となります。但し、

```
config.assets.debug = false
```

と、デバッグモードをオフにすれば連結状態になります。

サーバに初回リクエストにてアセットはコンパイル・キャッシュされ、304を返すようになります。マニフェスト記載のどれかが変更があると新しくコンパイルしてファイルを応答します。

デバッグモードは以下でも有効に出来ます。

```
<%= stylesheet_link_tag "application", :debug => true %>
<%= javascript_include_tag "application", :debug => true %>
```

パフォーマンス的にはデバッグモードは不利なので切り替えが提供されています

### Production環境

production環境ではFingerprintを利用します。プリコンパイル時にファイル内容に応じてMD5値が生成されてファイル名が更新されてディスクに格納されます。挿入された名前はマニフェスト、ヘルパーで補完されます。

```
<%= javascript_include_tag "application" %>
<%= stylesheet_link_tag "application" %>
```

にて


```
<script src="/assets/application-908e25f4bf641868d8683022a5b62f54.js" type="text/javascript"></script>
<link href="/assets/application-4dd5b109ee3439da54f5bdfd78a80473.css" media="screen" rel="stylesheet" type="text/css" />
```

となります。

config.assets.digest設定にてFingerprintのON/OFFを切り替えられます。production環境で有効、その他は無効になっています。通常はこのデフォルト設定は変更すべきではありません。キャッシュ制御がクライアント側で困難になるからです。

### Assetsのプリコンパイル

development環境のデフォルトでは動的にAssetファイルがコンパイルされますが、production環境のデフォルトでは高速化を図るため、事前にコンパイルして静的ファイルとして配置するようになっています。これをプリコンパイルといいます。Railsではrakeタスクとして配布されています。

コンパイルされたAssetはconfig.assets.prefixに設定されたディレクトリに格納されます。デフォルトはpublic/assetsディレクトリです。

デプロイサーバー上でのプリコンパイルも可能ですが、ファイル書き込み権限が無い等、rakeタスクを直接実行出来ない時はローカルでrakeタスクを実行してください。実行するrakeタスクは、以下のようになります。

```
bundle exec rake assets:precompile
```

プリコンパイルの実行速度を早めるためにconfig.assets.initialize_on_precompile設定をfalseに設定してRailsアプリケーションのロードをプリコンパイル時には部分的に行うようにすることが可能となります。Herokuではこの設定は必須となっています。

config.assets.initialize_on_precompile設定をfalseにした際は、ローカルでプリコンパイルが正しく実行出来るか確認してからでプロイして下さい。アプリケーションのオブジェクトやメソッドを参照していたりするとエラーが発生する場合があるので注意する必要があります。

Capistano(v2.8.0以降)にはプリコンパイル実行を制御するレシピが用意されています。以下の行をCapfileに追加してください。

```
load 'deploy/assets'
```

これによって、config.assets.prefixはshared/assetsディレクトリに設定されます。既にこのフォルダを利用していた場合は独自にrakeタスクを作成して下さい。

また、重要な点として、古いプリコンパイルされたAssetsファイルがページキャッシュ有効な間は正しく参照可能となるようにこのsharedフォルダはデプロイサーバー間で共有されている必要があります。

Assetsファイルをローカルでプリコンパイルする時は、デプロイ環境ではAsset関連のgemが必要なくなるので、以下のコマンドでassetsグループのgemを除外すると良いでしょう。

```
bundle install --without assets
```

プリコンパイルを行うファイルとして、application.js、application.cssとJS/CSSファイルではない全てのファイルが含まれます。もし、他にも独自のJS/CSSファイルを読み込みたい場合は以下のように追加設定する必要があります。

```
config.assets.precompile += ['admin.js', 'admin.css', 'swfObject.js']
```

この設定により、rakeタスクによるmanifest.ymlにはassetsファイルとそのFingerprintsの対応付けが行われ、ヘルパを呼び出すたびにSprocketsによるFingerprintへのマッピングリクエストを回避する事が出来ます。マニフェストファイルの例は以下のような感じです。

```
---
rails.png: rails-bd9ad5a560b5a3a7be0808c5cd76a798.png
jquery-ui.min.js: jquery-ui-7e33882a28fc84ad0e0e47e46cbf901c.min.js
jquery.min.js: jquery-8a50feed8d29566738ad005e19fe1c2d.min.js
application.js: application-3fdab497b8fb70d20cfc5495239dfc29.js
application.css: application-8af74128f904600e41a6e39241464e03.css
```

マニフェストファイルの配置場所はconfig.assets.prefixのルートディレクトリに配置されます。変更したい場合は以下にて設定出来ます。

```
config.assets.manifest = '/path/to/some/other/location'
```

Production環境でプリコンパイルされたファイルが見つからなければ、存在しないファイル名と共にSprockets::Helpers::RailsHelper::AssetPaths::AssetNotPrecompiledError の例外がRaiseされます。

#### サーバーの設定

プリコンパイルされたAssetファイルはファイルシステム上に静的ファイルとして配置され、Webサーバーより直接提供出来ます。Fingerprintingのメリットを十分に生かすため、以下のようにサーバー設定を行う事が推奨されます。

Apache:

```
<LocationMatch "^/assets/.*$">
  Header unset ETag
  FileETag None
  # RFC says only cache for 1 year
  ExpiresActive On
  ExpiresDefault "access plus 1 year"
</LocationMatch>
```

nginx:

```
location ~ ^/assets/ {
  expires 1y;
  add_header Cache-Control public;
 
  add_header ETag "";
  break;
}
```

プリコンパイル時、gzip圧縮(.gz)ファイルも同時に作成されます。WebサーバーではしばしばGzip圧縮転送の設定を行って転送量の軽減を図りますが、動的な圧縮だと、CPUの使用も懸念して圧縮率に妥協をすることがあります。この問題を解決するために事前に最小サイズとなるようにgzip圧縮ファイルを高圧縮率で作成した物を直接ディスクより提供する事でサーバー負荷を低減させるように設定する事もできます。

Nginxではgzip_staticを有効にする事により可能になります。設定は以下の通り。

```
location ~ ^/(assets)/  {
  root /path/to/public;
  gzip_static on; # to serve pre-gzipped version
  expires max;
  add_header Cache-Control public;
}
```

このディレクティブは大凡のコンパイル済みWebサーバーで利用出来ますが、出来ない場合はモジュールを追加してコンパイルする必要がある場合があります

```
./configure --with-http_gzip_static_module
```

Phusion Passengerと一緒にnginxをコンパイルする場合はこのオプションを指定する必要があります。

Apacheでも可能ですがやや困難です。トライしたいならググってください。

### 動的コンパイル(Live Compilation)

動的コンパイルをやりたい環境がある場合はSprocketsに全てのリクエストを投げる事も出来ます。以下のように設定します

```
config.assets.compile = true
```

Assetの初回アクセス時にコンパイルされdevelopment環境よりも上位環境ではキャッシュされてMD5ハッシュ化されたファイル名が使用されます。

SprocketsはCache-Controlヘッダにmax-age=31536000を設定してきます。これはサーバー、クライアント間でリクエストされたデータは1年間キャッシュしても良いという取り決めになります。この事によりブラウザ等でのローカルキャッシュや中間キャッシュを行うことができるようになり、サーバーへのリクエスト削減が期待出来ます。

しかしながら、動的コンパイルは多くのメモリやサーバ負荷となり得るのでお勧め出来ません。

もし、JavaScriptランタイムが導入されていないproduction環境のサーバーデプロイしたい場合は以下をGemfileに追加してください

```
group :production do
  gem 'therubyracer'
end
```

## パイプラインのカスタマイズ

### CSS圧縮

現在CSS圧縮エンジンとしてYUIがあります。YUIはCSSの最小化を行います。

```
config.assets.css_compressor = :yui
```

config.assets.compressはtrueにしておく必要があります。

### JavaScript圧縮

JS圧縮は:closure、:uglifierand、:yuiの3種類から選択出来ます。それぞれ、closure-compiler、uglifier、yui-compressorのgemが必要になります。

デフォルトでGemfileにはuglifierが指定されています。これはNodeJSで書かれたUglifierJSのRubyラッパーです。スペース削減やif文を3項演算子に変換する等JSの論理変換を行ってサイズ最小化が行われます。以下のように指定してください

```
config.assets.js_compressor = :uglifier
```

JS圧縮指定時もconfig.assets.compressは有効である必要があります。

uglifierにはJSランタイムであるExecJSが必要になります。MacOSXやWindowsにはインストールされています。サポートされているOSに関してはExecJSのドキュメントをチェックして下さい。

### 独自圧縮ロジックの定義

自分で圧縮ロジックを定義する事もできます。compressメソッドで変換結果をリターンして下さい。

```
class Transformer
  def compress(string)
    do_something_returning_a_string(string)
  end
end
```

利用するにはオブジェクトを指定します

```
config.assets.css_compressor = Transformer.new
```

### assetsパスの変更

Asset公開パスのデフォルトは /assets です。3.1にアップデートする際に既にこのURIが利用されていた等の際には以下にてパスを変更する事も出来ます

```
config.assets.prefix = "/some_other_path"
```

### X-Sendfileヘッダ

X-SendfileヘッダはWebサーバー自身がアプリケーションからのレスポンスをカスタマイズしてファイルをディスク上から直接読み込んで代わりにデータの受け渡しをします。デフォルトはオフですが、サポートされているなら有効にすることによりファイル提供の転送速度向上が期待できます。

Apache、nginxでサポートされており、以下をconfig/environments/production.rbに設定します

```
# config.action_dispatch.x_sendfile_header = "X-Sendfile" # for apache
# config.action_dispatch.x_sendfile_header = 'X-Accel-Redirect' # for nginx
```

既に存在しているアプリケーションのアップグレード時等に、production環境以外のX-Sendfileに対応していない環境化にこの設定をコピペしないように注意してください。

## キャッシュ

Sprocketsはdeveloment、production環境でRailsのデフォルトキャッシュストアを利用します。

デフォルトstore以外を指定出来る事が今後の課題です。

## Gemsを利用したAssetsの追加

AssetsはGemsの形式でも外部ソースを取り込めます。

jquery-rails gem等が良い例です。一般的なJSライブラリを取り込めます。このgemにはRails::Engineが継承されたクラスを含んでいて、中に含まれているapp/assetsやlib/assets、vendor/assets等のディレクトをSprocketsの検索パスとして追加されます。

以上。あと、3.1以前からのアップデートの方法等載っていました。興味のある人は原文を参照してみてください。
