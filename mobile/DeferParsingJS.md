---
layout: default
category: mobile
date: 2012-10-01
title: JavaScript の解析を遅延する
---
## JavaScript の解析を遅延する

原文：[https://developers.google.com/speed/docs/best-practices/mobile#DeferParsingJS](https://developers.google.com/speed/docs/best-practices/mobile#DeferParsingJS)

### Overview

ページを読み込むためには、ブラウザーはすべての`<script>`タグのコンテンツをパースしなければならない。これは、ページの読み込み時間に追加されます。ページのレンダリングに必要なJSを最小限にし、レンダリングに必要のないJSは実行の必要があるまでパースを遅らせることで、ページの初期読み込み時間を削減することが可能。

### Details

JavaScriptの遅延解析のテクニックはいくつかあり、最もシンプルで好まれている方法として、必要とされるまで[JavaScriptの読み込みを遅延させる](https://developers.google.com/speed/docs/best-practices/payload#DeferLoadingJS)テクニックがある。2番目の方法は[`<script async>`](/speed/rtt/PreferAsyncResources.html)属性を使用する方法で、ブラウザーUIスレッドのタスクが完了するまで遅延させることによって初期読み込みのブロックを防ぐ。もしこの2つのテクニックがどちらも適さないというのであれば、後述するモバイルアプリケーションで使用されるテクニックがある。

 モバイルアプリケーション構築の際、事前にアプリケーション必要なすべてのJSを読み込む可能性があり、それによってアプリケーションはオフライン時でも機能し続けることができる。モバイルGmailのようなものがこれにあてはまり、[JavaScriptをコメントとして読み込み](http://googlecode.blogspot.jp/2009/09/gmail-for-mobile-html5-series-reducing.html)、必要になったらeval()します。このアプローチにより、JSのパースをせずにすべてのJavaScriptを初期読み込みが可能だ。

コメントでコードを保存するのではなく文字列リテラルとしてコードを保存する方法もある。このテクニックを使うと、必要となったときに文字列リテラルに対して`eval()`することでJavaScriptをパースすればよい。これもまたパースなしにJavaScriptを読み込むことが可能。


`<script>`タグをページの最後に置くのは必ずしも最適な対応とはいえません。なぜならこのJavaScriptファイルのパースが完了するまでビジーインジケーターを表示するからです。ページ操作する前にユーザーがインジケーターが消えるまでその操作を待つかもしれないので、ブラウザーが示すページ読み込み完了の時間を削減することは重要なことです。

私達が2011年初頭に行った調査では、最近のモバイルデバイスに対して1KBのJavaScriptを追加すると1msのパース時間がページの読み込み時間にプラスされることがわかりました。つまり100KBのJSを含んだページの読み込み時間には100msの時間が追加されるのです。JavaScriptはページを訪問するたびにパースされるので、この追加のパースに要する時間はすべてのページで付加されることになります。それがネットワーク経由でダウンロードしたものであっても、ブラウザーキャッシュであろうとも、またはHTML5のオフラインキャッシュであろうとも。