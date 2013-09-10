---
layout: default
category: rtt
date: 2012-10-01
title: 非同期リソースを使用する
---
## 非同期リソースを使用する

原文：[https://developers.google.com/speed/docs/best-practices/rtt#PreferAsyncResources](https://developers.google.com/speed/docs/best-practices/rtt#PreferAsyncResources)

### Overview

リソースを非同期に読み込むことでページ読み込みのブロッキングを防ぐことができる。

### Details

基本的にブラウザーがScriptタグをパースするときは、そのスクリプトの後続のHTMLがレンダリング処理は、ダウンロード、パース、実行を待たなければならない。けれども非同期スクリプトの場合、ブラウザーはそのスクリプト以降のHTMLのパース・レンダリングをスクリプトの実行完了を待たずに続けることができる。スクリプトが非同期に読み込まれるとき、それは可能な限りすぐに読みこむが、実行はブラウザーのUIスレッドがビジーな状態（ページをレンダリングしているような）でなくなるまで、遅延される。

### Recommendations

アクセス解析のようなページの初期表示に必要のないJavaScriptは、非同期でに読み込まれるべきだ。ファーストビュー以降のページ構築に必要なスクリプトのような、ページ上であまり重要ではないコンテンツ表示に必要なスクリプトもまた、非同期で読み込んだほうがいいだろう。

__DOM要素でscriptを生成する __  
DOM要素でscriptを生成するすれば、最近のブラウザーで非同期に読みこむことができる。

{% highlight html %}
<script>
	var node = document.createElement('script');
	node.type = 'text/javascript';
	node.async = true;
	node.src = 'example.js';
	// Now insert the node into the DOM, perhaps using insertBefore()
</script>
{% endhighlight %}

async属性も指定しDOM要素でscriptを生成する方法の場合、Internet Explorer, Firefox, Chrome, Safariで非同期読み込みが可能だ。反対に、この文章執筆時点では、単純にHTMLの`<script>`タグにasync属性を追加した読み込みでは、ChromeまたはFirefox 3.6以上のブラウザでしか非同期読み込みできず、他のブラウザーはasync属性をサポートしていない。

__Google アナリティクスを非同期で読み込む__  
新しいバージョンのGoogle アナリティクススニペットも非同期読み込みに対応している。古いトラッキングスニペットを使用している場合は、[新しい非同期バージョンにアップデート](https://developers.google.com/analytics/devguides/collection/gajs/asyncTracking)すべきだ。

