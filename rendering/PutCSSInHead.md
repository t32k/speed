---
layout: default
category: rendering
date: 2012-10-01
title: CSS をドキュメントヘッドに含める
---
## CSS をドキュメントヘッドに含める

原文：[https://developers.google.com/speed/docs/best-practices/rendering#PutCSSInHead](https://developers.google.com/speed/docs/best-practices/rendering#PutCSSInHead)

### Overview

インラインスタイルと`<link>`はbodyからheadへ移動させるとレンダリングが改善される。

### Details

インラインスタイルと外部CSSをHTMLのbodyに記述することはレンダリングパフォーマンスに悪影響を与える。ブラウザーはすべての外部CSSが読み込まれるまでページのレンダリングをブロックする。
`<style>`タグで記述されたインラインスタイルはリフローを発生させるので、インラインスタイルも同様にページのheadで外部CSSを参照することが重要だ。スタイルシートが一番最初にダウンロード・パースされることによって、ブラウザーのプログレッシブレンダリングが可能となる。

### Recommendations

+ HTML4.01仕様書（[section12.3](http://www.w3.org/TR/html4/struct/links.html#h-12.3)）によると、外部CSSは常に `<head>` 内に `<link>` タグで記述すると書かれており、`@import`は使用しない。またスクリプトとスタイルシートを正しい順序で記述する。
+ `<style>` タグも `<head>` 部分に記述する。