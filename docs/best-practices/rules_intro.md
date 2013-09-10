---
layout: default
category: top
date: 2012-10-01
title: Webパフォーマンス ベストプラクティス - Make the Web Faster
---

### Webパフォーマンス ベストプラクティス

原文：[https://developers.google.com/speed/docs/best-practices/rules_intro](https://developers.google.com/speed/docs/best-practices/rules_intro)

Last updated: <time datetime="{{ site.time | date_to_xmlschema }}">{{ site.time | date_to_long_string }}</time>

翻訳：[@t32k](https://twitter.com/t32k)

WebページをPage Speedで調べるとルールに準拠していないものが提示される。このルールというのは、一般的にあなたが開発段階において取り入れるべきフロントエンドのベストプラクティスだ。あなたがPage Speedを使用しようとしまいと、私たちはこの各ルールについてのドキュメントを提供する（たぶんちょうど新しいサイトを開発中でテストする準備が整ってないだろう）。もちろん、これらのページはいつでも参照することができる。私たちはあなたの開発プロセスに取り入れてもらうために、このベストプラクティスを実装するための明確なティップスと提案を提供する。

#### パフォーマンス ベストプラクティスについて

Page Speedはクライアント側からの観点でパフォーマンスを評価し、一般的にページの読み込み時間を計測する。これはユーザーがページをリクエストした瞬間からブラウザによって完全にレンダリングされるまでの経過時間のことを指している。我々が紹介するベストプラクティスはページの読み込みにおける様々なステップをカバーしている。例えば、DNSの名前解決、TCPコネクションの確立、HTTPリクエストの伝達、リソースのダウンロード、キャッシュからのリソース読み込み、スクリプトのパースと実行、ページ上のオブジェクトのレンダリングに至るまでだ。基本的にPage SpeedをWebページで使用することで、それらのステップをちゃんとしているかどうか評価し、読み込みが完了するまでの時間を短縮させることができる。ベストプラクティスは異なる面からページ読み込み最適化カバーする6つのカテゴリに分類されている。

__キャッシュの最適化__ - アプリケーションのデータとロジックをネットワークから完全に分離させる

+ [ブラウザキャッシュを活用する](/speed/caching/LeverageBrowserCaching.html)
+ [プロキシーキャッシュを活用する](/speed/caching/LeverageProxyCaching.html)

__ラウンドトリップ時間の縮小化__ - リクエスト-レスポンスの一連のサイクルの回数を減らす

+ Minimize DNS lookups
+ [リダイレクトの回数を減らす](/speed/rtt/AvoidRedirects.html)
+ [誤りのあるリクエストを送信しない](/speed/rtt/AvoidBadRequests.html)
+ Combine external JavaScript
+ Combine external CSS
+ [CSSスプライトに画像をまとめる](/speed/rtt/SpriteImages.html)
+ [スタイルシートとスクリプトの順序を最適化する](/speed/rtt/PutStylesBeforeScripts.html)
+ Avoid document.write
+ [CSS @import を使用しない](/speed/rtt/AvoidCssImport.html)
+ [非同期リソースを使用する](/speed/rtt/PreferAsyncResources.html)
+ Parallelize downloads across hostnames

__リクエストのオーバーヘッドの縮小化 __ - アップロードサイズを減らす

+ Minimize request size
+ Serve static content from a cookieless domain

__読み込みサイズの減量__ - レスポンス、ダウンロード、キャッシュ化されたページのサイズを減らす

+ [圧縮を有効にする](/speed/payload/GzipCompression.html)
+ Remove unused CSS
+ [JavaScript を縮小する](/speed/payload/MinifyJS.html)
+ [CSS を縮小する](/speed/payload/MinifyCSS.html)
+ [HTML を縮小する](/speed/payload/MinifyHTML.html)
+ Defer loading of JavaScript
+ [画像を最適化する](/speed/payload/CompressImages.html)
+ [変更後のサイズで画像を提供する](/speed/payload/ScaleImages.html)
+ [重複するリソースは1つのURLから提供する](/speed/payload/duplicate_resources.html)


__ブラウザレンダリングの最適化__ - ブラウザのページレイアウト改善

+ Use efficient CSS selectors
+ Avoid CSS expressions
+ [CSS をドキュメントヘッドに含める](/speed/rendering/PutCSSInHead.html)
+ [画像のサイズを指定する](/speed/rendering/SpecifyImageDimensions.html)
+ [文字セットを指定する](/speed/rendering/SpecifyCharsetEarly.html)


__モバイルのための最適化__ - モバイル端末・ネットワーク特有のチューニング

+ [JavaScript の解析を遅延する](/speed/mobile/DeferParsingJS.html)
+ [リンク先ページのリダイレクトでキャッシュを利用できるようにする](/speed/mobile/PageRedirects.html)


#### フィードバックについて

どのようなフィードバックであろうとも送って頂ければ幸いだし、喜んでページに記載されている内容について説明したいと思う。もし、ベストプラクティスをよりよくする提案（もしくはドキュメントの改善）を持っているのであれば、[page-speed-discuss](http://groups.google.com/group/page-speed-discuss)に投稿してください。

#### その他のリソース

+ このサイトに書かれているベストプラクティスについてのさらなる詳細に関してはSteve Soudersの [ハイパフォーマンスWebサイト――高速サイトを実現する14のルール](https://www.oreilly.co.jp/books/9784873113616/)　と [続・ハイパフォーマンスWebサイト――ウェブ高速化のベストプラクティス](http://www.oreilly.co.jp/books/9784873114460/) を参照。
+ このサイトで書かれているサンプルコードの実行を確認したい場合は、同様に [14 Rules for Faster-Loading Web Sites](http://stevesouders.com/hpws/rules.php) を参照。
+ このサイトで書かれている実行可能なテストとブラウザの挙動の比較の指標に関しては [Browserscope](http://www.browserscope.org/) を参照。

---

### その他の記事

+ [gzip圧縮のしくみ](/speed/articles/gzip.html)
+ [Web高速化のために圧縮しよう](/speed/articles/use-compression.html)
