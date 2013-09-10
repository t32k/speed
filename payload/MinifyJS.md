---
layout: default
category: payload
date: 2012-10-01
title: JavaScript を縮小する
---
## JavaScript を縮小する

原文：[https://developers.google.com/speed/docs/best-practices/payload#MinifyJS](https://developers.google.com/speed/docs/best-practices/payload#MinifyJS)

### Overview

JavaScriptコードを圧縮させることはコードのバイト数を削減させることにつながり、ダウンロード・パース・実行時間を速めることが可能。

### Details

コードを縮小化するということは、余分なスペース、改行、インデントのような不要なバイトを取り除くということだ。JavaScriptをコンパクトに保つ利点はいくつかあり、まず最初にキャッシュさせたくないインライン・外部JSがある場合、ファイルサイズが小さければページのダウンロード毎に発生するレイテンシーを抑えることができる。2つ目に、縮小化は外部JS、HTML内に記述されたインラインJSの[圧縮](/speed/payload/GzipCompression.html)率をさらに高める。最後に、より小さなファイルは読み込みと起動が素早くなる。

[Closure Compiler](https://developers.google.com/closure/compiler/) や [JSMin](http://www.crockford.com/javascript/jsmin.html),  [YUI Compressor](http://developer.yahoo.com/yui/compressor/) などのいくつかのツールは無料でJavaScriptを縮小化できる。また、これらのツールを利用し縮小化・開発用ファイルのリネーム・プロダクションディレクトリに保存といったビルドプロセスを作ることも可能だ。4096バイト以上のJSファイルであれば縮小化することを推奨する。そして25バイト以上の削減（これ以下であればパフォーマンス対するメリットはない）が確認できるだろう。

Tip: JSファイルに対してPage Speedを起動させると自動的にClosure Complier(もし利用できるのであれば)か、JSMin（インラインJSに対して、Compilerが利用できない時）が起動し、縮小化されたファイルが [configurable directory](https://developers.google.com/speed/docs/insights/using_firefox#savefiles) に保存される。