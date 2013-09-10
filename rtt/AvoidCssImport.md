---
layout: default
category: rtt
date: 2012-10-01
title: CSS @import を使用しない
---
## CSS @import を使用しない

原文：[https://developers.google.com/speed/docs/best-practices/rtt#AvoidCssImport](https://developers.google.com/speed/docs/best-practices/rtt#AvoidCssImport)

### Overview

外部CSSファイル内でCSS @importを使用するとページ読み込みに遅延を発生させる。

### Details

CSS [@import](http://www.w3.org/TR/CSS2/cascade.html#at-import)を使用するとスタイルシートのインポートが可能となる。外部スタイルシート内で@importが使用されると、ブラウザーはスタイルシートを並列ダウンロードできない。これはページ読み込みに対して余計なラウンドトリップタイムを発生させる。例えば、`first.css`内に以下のコードが含まているとする：

{% highlight html %}
@import url("second.css")
{% endhighlight %}

ブラウザーはダウンロードする必要のある`second.css`を見つける前に、`first.css`のダウンロード、パース、実行をしてしまう。

### Recommendations

__@importの代わりに`<link>`タグを使用する __  
各スタイルシートの@importの代わりに`<link>`タグを使用する。これでブラウザーはスタイルシートを並列にダウンロードでき、結果的にページ読み込み時間を短縮できる。 

{% highlight html %}
<link rel="stylesheet" href="first.css">
<link rel="stylesheet" href="second.css">
{% endhighlight %}