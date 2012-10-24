---
layout: post
title: "i18n(Internationalization)"
date: 2012-10-22 13:03
comments: true
categories: [i18n]
---

Railsでi18nを使った多言語対応について調べたので、それについてまとめる。
今回は多言語対応の一般的な話をまとめる。

### 多言語対応について

ソフトウェアを多言語対応するときの工程として

* i18n(Internationalization:国際化)
* l10n(Localization:地域化)
* m17n(Multilingualization:多言語化)

の各ステップがある。
名前の由来は最初と最後の文字と間の文字数からきている。i18nだとInternationalizationの最初のiと最後のn、その間に18文字あることからきている。

「i18n」とは、ソフトウェアに技術的な変更を加えることなく、多言語、多地域に対応させる枠組みを作っておくことである。
i18nに対応すると次に各言語においての対応を実装していく必要がある。それが「l10n」で、特定の1言語で必要とされる言語特有の機能等を実装し、対応させることである。
多言語に渡ってl10nの対応をし、利用者の言語に合わせて切り替えて表示できる状態が「m17n」対応となり、多言語対応となる。

### Railsにおける多言語対応

Railsにはi18nの機能が標準でついており、利用することで多言語対応ができる。
詳しくは次回以降で記載する。

### 参考

* [多言語対応の問題と解決を考える](http://www.atmarkit.co.jp/fxml/rensai/xmlwomanabou11/learning-xml11.html)
* [Ruby on Rails Guides: Rails Internationalization (I18n) API](http://guides.rubyonrails.org/i18n.html)
