---
layout: post
title: "Web Storageの使い方"
author: aizawa1126
date: 2012-11-12 14:55
comments: true
categories: 
---
### Web Storageとは

Web Storageは、HTML5の周辺APIのひとつで、ブラウザにデータを保存するための仕組みです。データの保存・上書き・削除・全クリアなどの操作は、JavaScriptで行います。
Web StorageはCookieとよく似ていますが、Cookieに比べてはるかに大きな容量のデータをブラウザに保存できます。
Web Storageには、sessionStorageとlocalStorageの2種類のストレージが用意されています。どちらもキーと値をペアにしたデータリストをブラウザに保存するkey-value型のデータ保存形式である点は同じですが、データの有効期限などが異なります。

### CookieとsessionStorageとlocalStorageの差異

機能 | Cookie | sessionStorage | localStorage
-------------|--------------|---------
保存容量 | 4KB |  1オリジン当たり5MB(推奨) | 1オリジン当たり5MB(推奨)
データの有効期限 | 指定期限まで有効 | ウィンドウやタブを閉じるまで有効 | 永続的に有効
サーバーへのデータ送信 | 毎回自動送信 | 必要時のみ送信 | 必要時のみ送信
別ウィンドウでのデータ共有 | 可 | 不可 | 可

オリジン： プロトコル://ドメイン名:ポート番号 のこと

### Web Storageのメソッドと使い方

Web Storageで提供されているメソッドは、データの保存・取得・指定キーの値削除・全値クリアの4つです。メソッドはsessionStorageとlocalStorageで共通です。
```
//データの保存
setItem(key, value)

//データの取得
getItem(key)

//指定キーの値削除
removeItem(key)

//全値クリア
clear()
```
データの上書きをする場合はもう一度keyとvalueを保存します。メソッドの使い方は以下の通り。
```
var storage = localStorage;

//userIdキーに1を保存
storage.setItem('userId', '1');

//userIdキーに新たなvalueをセットし直せば上書き保存
storage.setItem('userId', '2');

//userIdキーの値を取得（2が返る）
storage.getItem('userId');

//userIdキーの値を削除
storage.removeItem('userId');

//ストレージにあるデータをすべてクリア
storage.clear();
```
