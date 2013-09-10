---
layout: default
category: rtt
date: 2012-10-01
title: スタイルシートとスクリプトの順序を最適化する
---
## スタイルシートとスクリプトの順序を最適化する

原文：[https://developers.google.com/speed/docs/best-practices/rtt#PutStylesBeforeScripts](https://developers.google.com/speed/docs/best-practices/rtt#PutStylesBeforeScripts)

### Overview

外部スタイルシートと外部とインラインスクリプトの順序を正しくすることは、ダウンロードの並列化を可能にし、レンダリングを早める。

### Details

JavaScriptコードはページのコンテンツ、レイアウトを変更できるため、ブラウザーはスクリプトがダウンロード・パース・実行が完了するまで、スクリプトタグ以降のコンテンツのレンダリングは遅延する。しかしながら、ラウンドトリップタイムに関してより重要なことは、多くのブラウザーはスクリプトの後に記述されたリソースのダウンロードをブロックする。反対に、もしJSファイルを参照した時に他のファイルがすでにダウンロード完了になっているのならば、他のリソースは並列ダウンロードされ、JSファイルもダウンロードされる。例えば、3つのスタイルシートと2つのスクリプトが以下のような順序で記述されているとする。

{% highlight html %}
<head>
	<link rel="stylesheet" type="text/css" href="stylesheet1.css" />
	<script type="text/javascript" src="scriptfile1.js" />
	<script type="text/javascript" src="scriptfile2.js" />
	<link rel="stylesheet" type="text/css" href="stylesheet2.css" />
	<link rel="stylesheet" type="text/css" href="stylesheet3.css" />
</head>
{% endhighlight %}

それぞれのリソースはダウンロードにきっちり100msかかり、ブラウザーは1つのホストに対して６つの同時接続が可能（詳細な情報に関しては、[Parallelize downloads across hostnames](https://developers.google.com/speed/docs/best-practices/rtt#ParallelizeDownloads) を参照）とし、キャッシュは空だと想定した場合、ダウンロードは以下のようになるだろう。

![](https://developers.google.com/speed/docs/insights/images/waterfall1.png)

後続の2つのスタイルシートはJSファイルのダウンロードが完了するまで待たなければならない。この総ダウンロード時間は２つのJSファイルと大きなCSSファイルをダウンロードしたようなものだ（この場合、100 ms + 100 ms + 100 ms = 300 ms）。単純にリソースの記述順序を変更してみる。

ダウンロード結果は以下のようになるだろう。

![](https://developers.google.com/speed/docs/insights/images/waterfall2.png)

総ダウンロード時間から100ms削減できた。ダウンロードするのに時間がかかりそうな大きなスタイルシートの場合はより多くの時間を節約できそうだ。

それゆえ、スタイルシートはドキュメントの上部に常に記述することはパフォーマンスにとって良いことなので、可能な限り、[headに含まれている](/speed/rendering/PutCSSInHead.html)かもしれない（もしくはドキュメント内に書かれている）いくつかのJSファイルをスタイルシートの後に記述することはダウンロード遅延を防ぐことにつながるので重要だ。

もうひとつ、次のようなインラインスクリプトがスタイルシートに続く場合に引き起こされる微妙な問題がある。

{% highlight html %}
<head>
	<link rel="stylesheet" type="text/css" href="stylesheet1.css" />
	<script type="text/javascript">
	 document.write("Hello world!");
	</script>
	<link rel="stylesheet" type="text/css" href="stylesheet2.css" />
	<link rel="stylesheet" type="text/css" href="stylesheet3.css" />
	<link rel="alternate" type="application/rss+xml" href="front.xml" title="Say hello" />
	<link rel="shortcut icon" type="image/x-icon" href="favicon.ico">
</head>
{% endhighlight %}

この場合、逆の問題が発生する。最初のスタイルシートはインラインスクリプトの実行をブロックし、次は他のリソースのダウンロードをブロックする。重ねて言うが、解決策はインラインスクリプトを可能な限り、すべてのリソースのあとに次のように記述する。

{% highlight html %}
<head>
	<link rel="stylesheet" type="text/css" href="stylesheet1.css" />
	<link rel="stylesheet" type="text/css" href="stylesheet2.css" />
	<link rel="stylesheet" type="text/css" href="stylesheet3.css" />
	<link rel="alternate" type="application/rss+xml" title="Say hello" href="front.xml" />
	<link rel="shortcut icon" type="image/x-icon" href="favicon.ico">
	<script type="text/javascript">
	   document.write("Hello world!");
	</script>
</head>
{% endhighlight %}


### Recommendations

__できる限り、外部スクリプトは外部スタイルシートの後に置く __  
ブラウザーはスタイルシートとスクリプトをそれらが出現する順番で実行する、もし、JSコードがCSSファイルに対して依存性がないのなら、JSファイルの前にCSSファイルを設置することができる。反対に、JSコードがCSSファイルに依存している、例えばJSコードで書かれたアウトプットをスタイルが必要とする場合などは不可能である。

__できる限り、インラインスクリプトは他のリソースの後に置く __  
すべてのリソースの後にインラインスクリプトを置くことは、他のダウンロードのブロッキングを防ぎ、プログレッシブレンダリングを可能とする。しかしながら、”他のリソース”が外部JSファイルで、インラインスクリプトに依存している場合は無理だろう。この場合、この場合CSSファイルのマにインラインスクリプトを移せば最適だ。
