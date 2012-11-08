---
layout: post
title: "ActionScript 3.0"
date: 2012-11-07 10:57
comments: true
categories: [ActionScript]
author: ntth
---

### ActionScript とは
ActionScript は Adobe Flash Player や Adobe AIR のランタイム環境用の開発言語です。
開発環境には Adobe Flash CS6(オーサリングツール)/Adobe Flex Builder(統合開発環境)/Adobe Flex(コマンドラインのコンパイラ)がありますが、
有償であったりコマンドラインでコンパイルしなければならないので、
ここでは無償の統合開発環境である FlashDevelop を使用しています。
ActionScriptで書かれたコードを上記環境でコンパイルすることにより、SWF(Shockwave Flash file)ファイルが生成されます。
このSWFファイルをウェブページなどに組み込めば、ランタイム環境で動作させることができます。

ActionScript には ActionScript 1.0/2.0/3.0 の各バージョンがあり、2.0からオブジェクト指向言語になっています。
ECMAScriptをベースに作られているため、Javascriptに似ており。また、オブジェクト指向言語になり、Javaにも似ている言語となっています。

### 文法

基本的なことと気になったことについて書いていきます。

#### 変数宣言

var 変数名:型 で宣言します。
{% codeblock %}
var num1:Number; //小数点付きまたは小数点なしの値を含むすべての数値
var num2:int; //整数
var num3:uint; //符号なし整数
var boo:Boolean; //true か false の2値
var str:String; //文字列
{% endcodeblock %}
などがあります。

#### 文字列

文字列はシングルクォートかダブルクォートを使って定義します。\n(改行)などの特殊文字を入れることもできます。
{% codeblock %}
var str:String = "kaigyou¥n"
{% endcodeblock %}

文字列を操作するメソッドも用意されており、Rubyみたいに使えます。
{% codeblock %}
var str:String = "FLASH";
trace(str.split(""));//traceを使用すると開発ツール上で実行したときに出力欄に表示される
//>>F,L,A,S,H
trace(str.length);//文字列の長さ
//>>5
trace([str, "TEST",].join("　"));//配列要素間を空白で結合
//>>FLASH TEST
{% endcodeblock %}
上記コードだけでは実行できないので、以下のようにコードを.asファイルに記載して実行(FlashDevelopではF5)します。
{% codeblock %}
package 
{
	import flash.display.Sprite;
	public class Main extends Sprite 
	{
		public function Main():void 
		{
			var str:String = "FLASH";
			trace(str.split(""));
		}
	}
}
{% endcodeblock %}

#### 配列

配列用の型があるので、Arrayで宣言します。
{% codeblock %}
var arr:Array;
arr = [a,b,c];
{% endcodeblock %}

配列も文字列と同じようにメソッドが用意されており、操作できます。
{% codeblock %}
var arr:Array = ['a','b','c'];
arr.push('d'); //末尾にデータ追加
trace(arr);
//>>[a,b,c,d]
arr.pop(); //末尾のデータ取り出し
trace(arr);
//>>[a,b,c]
{% endcodeblock %}

#### ハッシュ

Object型を使って、ハッシュを作ります。Object型はすべてのクラス定義の基本クラスです。
{% codeblock %}
var hash:Object = new Object();
hash["program"] = "ActionScript"
trace(hash["program"]);
//>>ActionScript
{% endcodeblock %}

#### ブロックスコープ

変数は関数単位で管理しており、ブロック変数として宣言したつもりでも関数スコープになっています。(withを使えば、実現できるらしいです。)
{% codeblock %}
//多重定義でコンパイルエラー
for ( var i:Number = 0; i < 1; i++) {
  i = 1;
}
for ( var i:String = 0; i < 1; i++) {
  i = 'a';
}
{% endcodeblock %}

## 例外

Javaのように例外をキャッチできます。
{% codeblock %}
try {
    //例外が検出したい処理
} catch (e:Error) {
    // 例外処理
}
{% endcodeblock %}

## クラス

以下のようにクラス定義します。
{% codeblock %}
package パッケージ名 { // パッケージ名省略可
    import パッケージ.クラス名;
    
    public class main(クラス名、仮にmain) extends 親クラス名
    {
        // 変数宣言
        アクセス修飾子 var プロパティ名:プロパティの型;
        
        // コンストラクタ
        アクセス修飾子 function main()//コンストラクタ名
        {
            処理1
        }
        
        // メソッド1
        アクセス修飾子 function メソッド名1(引数1:型 = デフォルト値):戻り値の型
        {
            処理2
        }
    }
}
{% endcodeblock %}

## 表示リスト

ActionScript 3.0 で構築されたアプリケーションには、表示リストと呼ばれるオブジェクトの階層があります。
{% codeblock %}
ステージ //Stageクラス
  - SWFファイルのメインクラスのインスタンス //自分で定義したクラス
    - 表示オブジェクト //Spriteクラス
    - 表示オブジェクトコンテナ //TextFieldなどの入れ物
      -(表示オブジェクト/表示オブジェクトコンテナ)
{% endcodeblock %}
* ステージ

表示オブジェクトの基本コンテナ、各アプリケーションには1つのStageオブジェクトがあり、この中に画面の表示オブジェクトがすべて含まれます。
ステージは、表示リスト階層の最上位にあたります。
それぞれのSWFファイルには関連するActionScriptクラスがあり、これがSWFファイルのメインクラスと呼ばれます。
SWFファイルのメインクラスは、Spriteクラスを拡張して定義します。
SWFファイルが Flash Player または Adobe AIR 上で開かれると、SWFファイルのメインクラスのコンストラクタ関数が呼ばれ、
作成されるインスタンスがStageオブジェクトに子として追加されます。

* 表示オブジェクト

ActionScript 3.0 では、アプリケーション内で表示される全てのエレメントタイプは、表示オブジェクトです。

* 表示オブジェクトコンテナ

表示オブジェクトコンテナは特殊な型の表示オブジェクトです。
表示オブジェクトコンテナ(単にコンテナともいいます)は、それ自体が表示オブジェクトコンテナ/表示オブジェクトを子オブジェクトに含むことができます。
ちなみに、ステージは表示オブジェクトコンテナです。
子オブジェクトに追加するには addChild関数を使います。

## Hello ActionScript
Flash上に「Hello ActionScript」と文字列を表示させます。
{% codeblock %}
package hello
{
	import flash.display.Sprite; //画面表示の基本クラスのインポート
	import flash.text.*; //テキスト系のクラス
	public class Main extends Sprite
	{
		public function Main (){//コンストラクタ
			var textField:TextField = new TextField();//入れ物確保
			textField.text = "Hello ActionScript";//文字列挿入
			addChild(textField);//textFieldをSpriteクラスに追加して表示
		}	
	}
}
{% endcodeblock %}



#### 参考

* [Adobe ActionScript 3.0 * Adobe Flash 用 Adobe ActionScript](http://help.adobe.com/ja_JP/ActionScript/3.0_ProgrammingAS3/) - 公式ドキュメント
* [読書メモ＋tips＋日記 : [Flash] ActionScript 3.0 基礎文法最速マスター](http://blog.livedoor.jp/takaaki_bb/archives/51374100.html)
* [ActionScript入門Wiki - トップページ](http://www40.atwiki.jp/spellbound/)