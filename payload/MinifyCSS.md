---
layout: default
category: payload
date: 2012-10-01
title: CSS を縮小する
---
## CSS を縮小する

原文：[https://developers.google.com/speed/docs/best-practices/payload#MinifyCSS](https://developers.google.com/speed/docs/best-practices/payload#MinifyCSS)

### Overview

CSSコードを縮小させることはコードのバイト数を削減させることにつながり、ダウンロード・パース・実行時間を速めることができる。

### Details

CSSの縮小化は、JavaScriptの縮小化と同様のメリットがある。ネットワークのレイテンシーを削減したり圧縮効率を高めたり、ブラウザーの読み込みと実行時間を速めてくれる。

例えば、[YUI Compressor](http://developer.yahoo.com/yui/compressor/) や [cssmin.js](http://www.phpied.com/cssmin-js/) のような、CSSを縮小化する無料のツールがいくつかある。

Tip: CSSファイルに対してPage Speedを起動させると自動的にcssmin.jsが起動し、縮小化されたファイルが [configurable directory](https://developers.google.com/speed/docs/insights/using_firefox#savefiles) に保存される。