---
layout: default
category: payload
date: 2012-10-01
title: HTML を縮小する
---
## HTML を縮小する

原文：[https://developers.google.com/speed/docs/best-practices/payload#MinifyHTML](https://developers.google.com/speed/docs/best-practices/payload#MinifyHTML)

### Overview

そのページに含まれるインラインのJS/CSSを含めてHTMLを圧縮させることはコードのバイト数を削減させることにつながり、ダウンロード・パース・実行時間を速めることができる。

### Details

HTMLの縮小化は、JS/CSSの縮小化と同様のメリットがあります。ネットワークのレイテンシーを削減したり圧縮率を高めたり、ブラウザーの読み込みと実行時間を速めてくれる。さらにHTMLには、しばしばインラインのJS`<script>`とCSS`<style>`も含まれているため、それらにも同様に圧縮のメリットがあります。

> *Note:* このルールは実験的であり、現在のバージョンでは厳格なHTMLの整合性というよりもサイズ減量にフォーカスしている。将来のバージョンでもこのような整合性もルールに追加する予定だ。現在の詳しい挙動に関しては [Page Speed wiki](http://code.google.com/p/page-speed/wiki/MinifyHtml) を参照。

*Tip:* HTMLファイルに対してPage Speedを起動させると自動的にHTML compactor（インラインのJavaScriptとCSSにはJSMinとcssmin.jsがそれぞれ適用）が起動し、縮小化されたファイルが [configurable directory](https://developers.google.com/speed/docs/insights/using_firefox#savefiles) に保存されます。