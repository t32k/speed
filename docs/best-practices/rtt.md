# Minimize round-trip times

Round-trip time (RTT) is the time it takes for a client to send a request and the server to send a response over the network, not including the time required for data transfer. That is, it includes the back-and-forth time on the wire, but excludes the time to fully download the transferred bytes (and is therefore unrelated to bandwidth). For example, for a browser to initiate a first-time connection with a web server, it must incur a minimum of 3 RTTs: 1 RTT for DNS name resolution; 1 RTT for TCP connection setup; and 1 RTT for the HTTP request and first byte of the HTTP response. Many web pages require dozens of RTTs.

RTTs vary from less than one millisecond on a LAN to over one second in the worst cases, e.g. a modem connection to a service hosted on a different continent from the user. For small download file sizes, such as a search results page, RTT is the major contributing factor to latency on "fast" (broadband) connections. Therefore, an important strategy for speeding up web page performance is to minimize the number of round trips that need to be made. Since the majority of those round trips consist of HTTP requests and responses, it's especially important to minimize the number of requests that the client needs to make and to parallelize them as much as possible.

+ Minimize DNS lookups
+ [リダイレクトの回数を減らす](#リダイレクトの回数を減らす)
+ Avoid bad requests
+ Combine external JavaScript
+ Combine external CSS
+ [CSSスプライトに画像をまとめる](#CSSスプライトに画像をまとめる)
+ Optimize the order of styles and scripts
+ Avoid document.write
+ CSS @import を使用しない
+ Prefer asynchronous resources
+ Parallelize downloads across hostnames


## Minimize DNS lookups

### 概要

Reducing the number of unique hostnames from which resources are served cuts down on the number of DNS resolutions that the browser has to make, and therefore, RTT delays.

### 詳細

Before a browser can establish a network connection to a web server, it must resolve the DNS name of the web server to an IP address. Since DNS resolutions can be cached by the client's browser and operating system, if a valid record is still available in the client's cache, there is no latency introduced. However, if the client needs to perform a DNS lookup over the network, the latency can vary greatly depending on the proximity of a DNS name server that can provide a valid response. All ISPs have DNS servers which cache name-IP mappings from authoritative name servers; however, if the caching DNS server's record has expired, and needs to be refreshed, it may need to traverse several nodes in the DNS serving hierarchy, sometimes around the globe, to find an authoritative server. If the DNS resolvers are under load, they can queue DNS resolution requests, which further adds to the latency. In other words, in theory, DNS resolution takes 1 RTT to complete, but in practice, the latency can vary significantly due to DNS resolver queuing delays. It's therefore important to reduce DNS lookups more than any other kinds of requests.

The validity of a DNS record is determined by the time-to-live (TTL) value set by its primary authoritative server; many network administrators set the TTL to very low (between 5 minutes and 24 hours) to allow for quick updates in case network traffic needs to be shifted around. (However, many DNS caches, including browsers, are "TTL disobeyers" and keep the cached record for longer than instructed by the origin server, up to 30 minutes in some cases.)  There are a number of ways to mitigate DNS lookup time — such as increasing your DNS records' time-to-live setting, minimizing CNAME records (which require additional lookups), replicating your name servers in multiple regions, and so on — but these go beyond the scope of web application development, and may not be feasible given your site's network traffic management requirements.

Instead, the best way to limit DNS-lookup latency from your application is to minimize the number of different DNS lookups that the client needs to make, especially lookups that delay the initial loading of the page. The way to do that is to minimize the number of different hostnames from which resources need to be downloaded. However, because there are benefits from using multiple hostnames to induce parallel downloads, this depends somewhat on the number of resources served per page. The optimal number is somewhere between 1 and 5 hosts (1 main host plus 4 hosts on which to parallelize cacheable resources). As a rule of thumb, you shouldn't use more than 1 host for fewer than 6 resources; fewer than 2 resources on a single host is especially wasteful. It should never be necessary to use more than 5 hosts (not counting hosts serving resources over which you have no control, such as ads).

### 推奨

___Use URL paths instead of hostnames wherever possible.__  
If you host multiple properties on the same domain, assign those properties to URL paths rather than separate hostnames. For example, instead of hosting your developer site on developer.example.com, host it on www.example.com/developer. Unless there are good technical reasons to use different hostnames, e.g. to implement DNS-based traffic load-balancing policies, there's no advantage in using a hostname over a URL path to encode a product name. In fact, the latency improvements of consolidating properties on a single hostname can actually enhance user experience: users can link between properties in a single browsing session with no additional DNS lookup penalty. In addition, reusing domains allows the browser to reuse TCP connections more frequently, further reducing the number of round trip times. If one property receives a lot of traffic, reusing that hostname for other properties can also increase the DNS cache hit rate for first-time visits, since the likelihood of a valid mapping already existing on a local caching server is higher. 

___Serve early-loaded JavaScript files from the same hostname as the main document.___  
It's especially important to minimize lookups in the "critical path". We define the critical path as the code and resources required to render the initial view of a web page. In particular, external JavaScript files that you own and that are loaded from the document head, or early in the document body, should be served from the same host as the main document. Most browsers block other downloads and rendering while JavaScript files are being downloaded, parsed and executed. Adding DNS lookup time to this process further delays page load time. If it's not possible to serve those files from the same hostname as the containing document, defer loading them, if possible. The exception is for JS files that are shared by pages served off multiple domains: in this case, serving the file from a unique URL to increase the cache hit rate may outweigh the DNS lookup overhead.


## リダイレクトの回数を減らす

### 概要

HTTPリダイレクト最小限にすることでユーザーの待ち時間と余計なRTTを削除することができる。

### 詳細

アプリケーションにおいて時折、ブラウザーをあるURLからほかのURLへ転送させる必要があるだろう。これにはWebアプリケーションがリダイレクトを発行するいくつかの理由がある。

+ 移動してしまったリソースの新しい場所を示すため
+ インプレション数とクリック数とリファラーログをトラッキングするため
+ 複数のドメインを予約するため、ユーザーフレンドリーまたは無意味なドメインを使用するため、URLのミススペル、打ち間違いをキャッチするため。
+ 異なるサイトまたはアプリケーションをつなぐため、異なる国コードとトップレベルドメインをつなぐため、異なるプロトコルHTTPからHTTPS）をつなぐため、異なるセキュリティポリシー（例：認証されていないページと認証されたページ）をつなぐためなど
+ ブラウザーがコンテンツにアクセスできるようにするために、URLの末尾のスラッシュ追加のため

いかなる理由であろうとも、リダイレクトは余計なHTTPリクエストーレスポンスサイクルとラウンドトリップタイムを発生させるトリガーとなる。あなたのアプリケーションでリダイレクトを最小限にすることは非常に重要だ（とりわけ、ホームページのような起動画面で必要なリソースは）。リダイレクトをする最適な方法は、完全に技術的にそれができないと無理なケースで、その他の解決方法も見つからない場合以外はリダイレクトを制限することだ。

### 推奨

__不要なリダイレクトは取り除く __  
単純に不要なリダイレクトを取り除くための戦略は以下：

+ ほかのURLへリダイレクトするページのURLを参照しない。あなたのアプリケーションはリソースのロケーションに変更があるたびにURLのリファレンスをアップデートする仕組みが必要となる。
+ リソースを取得するのに1回以上のリダイレクトを使用しない。例えば、Cというターゲットページがあり、2つの違うスタート地点のA,Bというページがある場合、AとB両方とも、直接Cにリダイレクトすべきだ。Aを中間のBへリダイレクトする必要はない。
+ 実際のサーバーコンテンツを提供するのではなく、リダイレクトを発行する余計なドメインを削減する。ときどき、ドメイン名の予約と間違ったユーザー入力（ミススペル、ミスタイプURL）のために複数のドメインにリダイレクトしたい誘惑にかられる。しかし、もしユーザーを複数のURLからあなたのサイトをたどり着くように訓練するのなら、ドメイン占拠者の手からあなたの名前のバリエーションのドメインを守るために新しいドメインを買い続けるような悪循環を精算することができる。

__ユーザー入力したURLのためにサーバーリライトを使用する__  
多くの Webサーバーは内部リライトをサポートしている。これは、ある URLから別のURLにマッピングが可能だということを意味している。クライントで間違ったURLのリクエストが来た場合、サーバーは自動的に正しいものに変更し、リダイレクトなしにリソースを提供する。あなたがコントロールできないURLをキャッチするためにそれを利用すべきだ。リライトを簡単なURLリファレンス更新のために使用するべきではない。あなたは常に1つのURLで1つのリソースを参照すべきである。また可能であればキャッシュ可能なリソースのためにそれらを使用することは避ける。ディレクトリ名の最後に末尾スラッシュを自動追加するのはユーザー入力したURLに良い候補を知らせるのでリライトの仕組みで良い見本と言える。

__バックグラウンドでトラフィックを追跡する__  
様々な情報をトラフィックするために、いくつかのサイトは集約的なサーバーでページのすべての情報を収集するために媒介的なリダイレクトを使用する。しかしながら、このようなリダイレクトはページ遷移の間に余計なレイテンシーを発生させてしまうため、リダイレクトは避け、ページビューを計測する違う方法を検討するのが良い。非同期にページビューを記録する一般的な方法は、ユーザーがページを読み込んだときにログサーバーに通知し、JavaScriptスニペット（または*onload*イベントハンドラなど）をターゲット·ページの下部に含めることである。最も一般的な方法としてはサーバーに対してビーコンをリクエストすることで、ビーコンリソースのURLパラメータとしてすべてのデータをエンコードしたものを付加する。HTTPレスポンスを非常に軽微なものに維持できるため、ビーコンのリクエストは1x1ピクセルの透明画像を選択するのがよい。さらに言えば、そのビーコンは1x1のGIFよりもわずかに小さい204レスポンス（"no content"）を返すとよい。

これはつまらない例だが、www.example.com/logger はログサーバーでbeacon.gifがリクエストされる画像としている。現在のページのURLと（もし存在すれば）リファラーのページのURLがパラメーターとして渡される。

```html
<script type="text/javascript">
	 var thisPage = location.href;
	 var referringPage = (document.referrer) ? document.referrer : "none";
	 var beacon = new Image();
	 beacon.src = "http://www.example.com/logger/beacon.gif?page=" + encodeURI(thisPage)
	 + "&ref=" + encodeURI(referringPage);
</script>
```

このビーコンは、他の実際にページコンテンツをレンダリングするのに必要なHTTPリクエストと教護しないように、ページのHTMLの最下部に含めるべきだ。このように、このリクエストはユーザーがページを訪れた時に、追加の待ち時間を発生させずに生成される。

__JavaScriptまたはmetaリダイレクトではなく、HTTPリダイレクトをする __  
リダイレクトを発行するいくつかの方法がある：

+ サーバーサイド：Webサーバーに対して、300　HTTPレスポンスコード（一般的なのは301"moved permanently"または302 　"found"/"moved temporarily"）を新しいURLの`Location`ヘッダーに発行するように設定する。
+ クライアントサイド：metaタグの`http-equiv="refresh"`属性を使用するか、JavaScriptの`window.location`オブジェクト（の`replace`メソッド）をHTMLドキュメントの上部で使用する。

リダイレクトを使用しなければならないのなら、クライアントサイドの方法よりもサーバーサイドの方法を選択したほうが良い。ブラウザーはmetaまたはJavaScriptによるリダイレクトより、HTTPリダイレクトをハンドリングしやすい。例えば、301または302リダイレクトはHTMLドキュメントがパースされる前に即座に処理されるが、JSリダイレクトはブラウザーのパース処理のレイテンシーの影響を受けてしまう。

加えて、HTTP/1.1の仕様書によれば、301と302レスポンスはブラウザーによってキャッシュが可能だ。これはリソース自体がキャッシュ不可能であっても、ブラウザーは少なくとも、ローカルキャッシュ内の正しいURLを探せることを意味している。301レスポンスは特別な指定がないかぎりデフォルトでキャッシュ可能である。302レスポンスをキャッシュ可能にするためには、あなたはWebサーバーに対して、`Expires`または`Cache-Control max-age`ヘッダーを設定(詳細は[ブラウザキャッシュを活用する](/speed/caching/LeverageBrowserCaching.html)を参照)
する。ここでの注意点は、多くのブラウザは、実際に仕様どおりではないので、301、302レスポンス両方ともキャッシュしないかもしれない。準拠および非準拠ブラウザのリストについては[Browserscope](http://www.browserscope.org/)を参照。

### 事例

[Google Analytics](http://www.google.com/analytics)アカウントオーナーによって所有されているWebページで、インバウンド内部、およびアウトバウンドトラフィックを追跡するために、画像ビーコン方式を採用しています。アカウントオーナーは`trackPageview（）`関数を定義し、Webページ内に外部JavaScriptファイルへの参照を埋め込む。HTMLドキュメントのbody下部にはユーザーが訪問した時にコールする、JavaScriptスニペットが含まれている。`trackPageview() `関数は__utm.gifという複数のパラメータつきのURLの1x1ピクセル画像を生成する。このパラメーターはページURL、リファラー、ブラウザ環境、ユーザー地域などのような情報を明示する。Analyticsサーバーがリクエストを受信したときに、情報を記録され、アカウントオーナーがレポーティングサイトにサインインしたときに提供される。

### その他のリソース
+ トラッキングコードの詳細に関しては [Google Analytics Developer Docs](http://code.google.com/apis/analytics/docs/index.html) を参照
+ Apache [URL Rewriting Guide](http://apache.org/docs/2.2/rewrite/) では、`mod_rewrite`を用いた内部リライトについて議論されている


## Avoid bad requests


### 概要

Removing "broken links", or requests that result in 404/410 errors, avoids wasteful requests.

### 詳細

As your website changes over time, it's inevitable that resources will be moved and deleted. If you don't update your frontend code accordingly, the server will issue 404 "Not found" or 410 "Gone" responses. These are wasteful, unnecessary requests that lead to a bad user experience and make your site look unprofessional. And if such requests are for resources that can block subsequent browser processing, such as JS or CSS files, they can virtually "crash" your site. In the short term, you should scan your site for such links with a link checking tool, such as the crawl errors tool in Google's Webmaster Tools, and fix them. Long term, your application should have a way of updating URL references whenever resources change their location.

### 推奨

__Avoid using redirects to handle broken links.__  
Wherever possible, you should update the links to resources that have moved, or delete those links if the resources have been removed. Avoid using HTTP redirects to send users to the requested resources, or to serve a substitute "suggestion" page. As described above, redirects also slow down your site, and are better to avoid as much as possible.


## Combine external JavaScript

### Overview

Combining external scripts into as few files as possible cuts down on RTTs and delays in downloading other resources.

### Details

Good front-end developers build web applications in modular, reusable components. While partitioning code into modular software components is a good engineering practice, importing modules into an HTML page one at a time can drastically increase page load time. First, for clients with an empty cache, the browser must issue an HTTP request for each resource, and incur the associated round trip times. Secondly, most browsers prevent the rest of the page from from being loaded while a JavaScript file is being downloaded and parsed. (For a list of which browsers do and do not support parallel JS downloads, see Browserscope.)

![fig]()

Here is an example of the download profile of an HTML file containing requests for 13 different .js files from the same domain; the screen shot is taken from Firebug's Net panel over a DSL high-speed connection with Firefox 3.0+:

![fig]()

All files are downloaded serially, and take a total of 4.46 seconds to complete. Now here is the the profile for the same document, with the same 13 files collapsed into 2 files:

The same 729 kilobytes now take only 1.87 seconds to download. If your site contains many JavaScript files, combining them into fewer output files can dramatically reduce latency.

However, there are other factors that come into play to determine the optimal number of files to be served. First, it's important also to defer loading JS code that is not needed at a page's startup. Secondly, some code may have different versioning needs, in which case you will want to separate it out into files. Finally, you might have to serve JS from domains that you don't control, such as tracking scripts or ad scripts. We recommend a maximum of 3, but preferably 2, JS files.
It often makes sense to use many different JavaScript files during the development cycle, and then bundle those JavaScript files together as part of your deployment process. See below for recommended ways of partitioning your files. You would also need to update all of your pages to refer to the bundled files as part of the deployment process. 

### Recommendations

__Partition files optimally.__  
Here are some rules of thumb for combining your JavaScript files in production:

+ Partition the JavaScript into 2 files: one JS containing the minimal code needed to render the page at startup; and one JS file containing the code that isn't needed until the page load has completed.
+ Serve as few JavaScript files from the document `<head>` as possible, and keep the size of those files to a minimum.
+ Serve JavaScript of a rarely visited component in its own file. Serve the file only when that component is requested by a user.
+ For small bits of JavaScript code that shouldn't be cached, consider inlining that JavaScript in the HTML page itself.


__Position scripts correctly in the document head.__  
Whether a script is external or inline, it's beneficial to position it in the correct order with respect to other elements, to maximize parallel downloads.


## Combine external CSS

### Overview

Combining external stylesheets into as few files as possible cuts down on RTTs and delays in downloading other resources.

### Details

As with external JavaScript, multiple external CSS files incurs additional RTT overhead. If your site contains many CSS files, combining them into fewer output files can reduce latency. We recommend a maximum of 3, but preferably 2, CSS files.

It often makes sense to use many different CSS files during the development cycle, and then bundle those CSS files together as part of your deployment process. See below for recommended ways of partitioning your files. You would also need to update all of your pages to refer to the bundled files as part of the deployment process.

### Recommendations

__Partition files optimally.__  
Here are some rules of thumb for combining your CSS files in production:

+ Partition the CSS into 2 files each: one CSS file containing the minimal code needed to render the page at startup; and one CSS file containing the code that isn't needed until the page load has completed.
+ Serve CSS of a rarely visited component in its own file. Serve the file only when that component is requested by a user.
+ For CSS that shouldn't be cached, consider inlining it.
+ Don't use CSS @import from a CSS file.

__Position stylesheets correctly in the document head.__  
It's beneficial to position references to external CSS in the correct order with respect to scripts, to enable parallel downloads.


## CSSスプライトに画像をまとめる

### 概要

CSSスプライトで画像をできる限りまとめることはリソースのダウンロードする際のラウンドトリップの回数を削減することだけでなく、リクエストのオーバーヘッドも削減し、なおかつトータルのダウンロードバイト数も削減できる。

### 詳細

JavaScript, CSS同様に複数の画像をダウンロードすることは余計なラウンドトリップを発生させるので、多くの画像を含むサイトはそれらをまとめ、少数のファイルにすることでレイテンシーを削減することができる。

### 推奨

__一緒に読み込まれる画像をスプライトする __  
同じページ上で読み込まれる、または常に一緒に読み込まれる画像をまとめる。例えば、すべてのページで読み込まれるアイコンセットのようなものはスプライトすべきだ。動的に生成されるようなプロフィール写真や頻繁に更新される画像はスプライトの候補としてふさわしくない。

__GIF/PNG画像をまずスプライトする __  
GIF/PNGはロスレス圧縮画像なのでスプライト化による画質の劣化はない。

__小さな画像をまずスプライトする __  
リクエスト毎に必ず[リクエスオーバーヘッド](https://developers.google.com/speed/docs/best-practices/request)が発生するので、小さな画像をダウンロードすることでリクエストオーバーヘッドによる時間が読み込み時間の大半となってしまう。小さな画像はスプライトでまとめることによって、画像ごとに発生するオーバーヘッドも1回のオーバーヘッドだけで済ませることができる。

__キャッシュ可能な画像をスプライトする __  
スプライト化した画像に[キャッシュを効かせる](/speed/caching/LeverageBrowserCaching.html)ということは、一度ブラウザーで読みこめば、再読み込みしなくても良いことを意味する。

__Spriteサービスを利用する__  
[Sprite Me](http://spriteme.org/)のようなサービスを利用すれば簡単にCSSスプライトを実装できる。

__スプライト画像の空白を最小限にする __  
画像を表示するためにブラウザーは画像に対して解凍とデコードをする。デコード化された画像サイズは画像のピクセル数に比例するので、スプライト画像の空白は画像サイズに対してインパクトを与えないかもしれないが、表示に使われないスプライト画像のピクセルがメモリの使用量増加させてしまう。これはブラウザの応答性を低くする可能性がある。

__似たようなカラーパレットの画像をスプライトする __  
256色以上のスプライト画像はパレットタイプカラーの代わりにPNG trueカラータイプになる。これはスプライト画像のサイズ増加を意味し、最適なスプライトは同じ256色カラーパレット内の画像をまとめることだ。もし、カラーにばらつきがあるのなら、256色に減色してからスプライトすることを考慮する。



## CSS @import を使用しない

### 概要

外部CSSファイル内でCSS @importを使用するとページ読み込みに遅延を発生させる。

### 詳細

CSS [@import](http://www.w3.org/TR/CSS2/cascade.html#at-import)を使用するとスタイルシートのインポートが可能となる。外部スタイルシート内で@importが使用されると、ブラウザーはスタイルシートを並列ダウンロードできない。これはページ読み込みに対して余計なラウンドトリップタイムを発生させる。例えば、`first.css`内に以下のコードが含まているとする：

{% highlight html %}
@import url("second.css")
{% endhighlight %}

ブラウザーはダウンロードする必要のある`second.css`を見つける前に、`first.css`のダウンロード、パース、実行をしてしまう。

### 推奨

__@importの代わりに`<link>`タグを使用する __  
各スタイルシートの@importの代わりに`<link>`タグを使用する。これでブラウザーはスタイルシートを並列にダウンロードでき、結果的にページ読み込み時間を短縮できる。 

```html
<link rel="stylesheet" href="first.css">
<link rel="stylesheet" href="second.css">
```



## 非同期リソースを使用する

### 概要

リソースを非同期に読み込むことでページ読み込みのブロッキングを防ぐことができる。

### 詳細

基本的にブラウザーがScriptタグをパースするときは、そのスクリプトの後続のHTMLがレンダリング処理は、ダウンロード、パース、実行を待たなければならない。けれども非同期スクリプトの場合、ブラウザーはそのスクリプト以降のHTMLのパース・レンダリングをスクリプトの実行完了を待たずに続けることができる。スクリプトが非同期に読み込まれるとき、それは可能な限りすぐに読みこむが、実行はブラウザーのUIスレッドがビジーな状態（ページをレンダリングしているような）でなくなるまで、遅延される。

### 推奨

アクセス解析のようなページの初期表示に必要のないJavaScriptは、非同期でに読み込まれるべきだ。ファーストビュー以降のページ構築に必要なスクリプトのような、ページ上であまり重要ではないコンテンツ表示に必要なスクリプトもまた、非同期で読み込んだほうがいいだろう。

__DOM要素でscriptを生成する __  
DOM要素でscriptを生成するすれば、最近のブラウザーで非同期に読みこむことができる。

```html
<script>
	var node = document.createElement('script');
	node.type = 'text/javascript';
	node.async = true;
	node.src = 'example.js';
	// Now insert the node into the DOM, perhaps using insertBefore()
</script>
```

async属性も指定しDOM要素でscriptを生成する方法の場合、Internet Explorer, Firefox, Chrome, Safariで非同期読み込みが可能だ。反対に、この文章執筆時点では、単純にHTMLの`<script>`タグにasync属性を追加した読み込みでは、ChromeまたはFirefox 3.6以上のブラウザでしか非同期読み込みできず、他のブラウザーはasync属性をサポートしていない。

__Google アナリティクスを非同期で読み込む__  
新しいバージョンのGoogle アナリティクススニペットも非同期読み込みに対応している。古いトラッキングスニペットを使用している場合は、[新しい非同期バージョンにアップデート](https://developers.google.com/analytics/devguides/collection/gajs/asyncTracking)すべきだ。



## 誤りのあるリクエストを送信しない

### 概要

壊れたリンクまたは404/410エラーを返すリクエストを取り除き、無駄なリクエストを避ける。

### 詳細

Webサイトは時間とともに変化するものなので、リソースのURLが変更になったり削除されたりすることは避けられないことだ。それにともなってフロントエンドのコードもアップデートしなければ、サーバーは404 Not Foundもしくは410 Goneレスポンスを返すことになるだろう。これらは無駄であり、悪いユーザー体験を引き起こし、素人くさい印象を与える不要なリクエストである。JSやCSSファイルのようなリソースをリクエストする場合にこのような状態になるとブラウザの処理をブロックすることになる、事実上サイトをクラッシュさせているようなものだ。短期的には、[Googleウェブマスターツール](http://www.google.com/webmasters/)のクロールエラーのようなものでリンクをチェックすべきです。長期的には、アプリケーションのリソースのロケーションが変更になるたびに、URLの参照先を更新すべきだろう。

### 推奨

__壊れたリンクのためにリダイレクトを使用するのは避ける __  
可能な限り、リソースが引越ししている場合はリンクを更新し、削除されている場合はリンクも削除すべきである。ユーザーに対してリクエストしたリソースにリダイレクトしたり、代替ページを提供するのは避けるべきである。前述のとおり、リダイレクトはサイトを遅くするので可能な限り避けたほうが賢明だ。




## スタイルシートとスクリプトの順序を最適化する

### 概要

外部スタイルシートと外部とインラインスクリプトの順序を正しくすることは、ダウンロードの並列化を可能にし、レンダリングを早める。

### 詳細

JavaScriptコードはページのコンテンツ、レイアウトを変更できるため、ブラウザーはスクリプトがダウンロード・パース・実行が完了するまで、スクリプトタグ以降のコンテンツのレンダリングは遅延する。しかしながら、ラウンドトリップタイムに関してより重要なことは、多くのブラウザーはスクリプトの後に記述されたリソースのダウンロードをブロックする。反対に、もしJSファイルを参照した時に他のファイルがすでにダウンロード完了になっているのならば、他のリソースは並列ダウンロードされ、JSファイルもダウンロードされる。例えば、3つのスタイルシートと2つのスクリプトが以下のような順序で記述されているとする。

```html
<head>
	<link rel="stylesheet" type="text/css" href="stylesheet1.css" />
	<script type="text/javascript" src="scriptfile1.js" />
	<script type="text/javascript" src="scriptfile2.js" />
	<link rel="stylesheet" type="text/css" href="stylesheet2.css" />
	<link rel="stylesheet" type="text/css" href="stylesheet3.css" />
</head>
```

それぞれのリソースはダウンロードにきっちり100msかかり、ブラウザーは1つのホストに対して６つの同時接続が可能（詳細な情報に関しては、[Parallelize downloads across hostnames](https://developers.google.com/speed/docs/best-practices/rtt#ParallelizeDownloads) を参照）とし、キャッシュは空だと想定した場合、ダウンロードは以下のようになるだろう。

![](https://developers.google.com/speed/docs/insights/images/waterfall1.png)

後続の2つのスタイルシートはJSファイルのダウンロードが完了するまで待たなければならない。この総ダウンロード時間は２つのJSファイルと大きなCSSファイルをダウンロードしたようなものだ（この場合、100 ms + 100 ms + 100 ms = 300 ms）。単純にリソースの記述順序を変更してみる。

ダウンロード結果は以下のようになるだろう。

![](https://developers.google.com/speed/docs/insights/images/waterfall2.png)

総ダウンロード時間から100ms削減できた。ダウンロードするのに時間がかかりそうな大きなスタイルシートの場合はより多くの時間を節約できそうだ。

それゆえ、スタイルシートはドキュメントの上部に常に記述することはパフォーマンスにとって良いことなので、可能な限り、[headに含まれている](/speed/rendering/PutCSSInHead.html)かもしれない（もしくはドキュメント内に書かれている）いくつかのJSファイルをスタイルシートの後に記述することはダウンロード遅延を防ぐことにつながるので重要だ。

もうひとつ、次のようなインラインスクリプトがスタイルシートに続く場合に引き起こされる微妙な問題がある。

```html
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
```

この場合、逆の問題が発生する。最初のスタイルシートはインラインスクリプトの実行をブロックし、次は他のリソースのダウンロードをブロックする。重ねて言うが、解決策はインラインスクリプトを可能な限り、すべてのリソースのあとに次のように記述する。

```html
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
```

### 推奨

__できる限り、外部スクリプトは外部スタイルシートの後に置く __  
ブラウザーはスタイルシートとスクリプトをそれらが出現する順番で実行する、もし、JSコードがCSSファイルに対して依存性がないのなら、JSファイルの前にCSSファイルを設置することができる。反対に、JSコードがCSSファイルに依存している、例えばJSコードで書かれたアウトプットをスタイルが必要とする場合などは不可能である。

__できる限り、インラインスクリプトは他のリソースの後に置く __  
すべてのリソースの後にインラインスクリプトを置くことは、他のダウンロードのブロッキングを防ぎ、プログレッシブレンダリングを可能とする。しかしながら、”他のリソース”が外部JSファイルで、インラインスクリプトに依存している場合は無理だろう。この場合、この場合CSSファイルのマにインラインスクリプトを移せば最適だ。

--

原文：https://developers.google.com/speed/docs/best-practices/rtt