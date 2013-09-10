---
layout: default
category: rendering
date: 2012-10-01
title: 文字セットを指定する
---
## 文字セットを指定する

原文：[https://developers.google.com/speed/docs/best-practices/rendering#SpecifyCharsetEarly](https://developers.google.com/speed/docs/best-practices/rendering#SpecifyCharsetEarly)


### Overview

HTTPレスポンスヘッダーで文字セットを明示することは、HTMLパース・スクリプト実行を早めてくれる。

### Details

HTMLドキュメントは、文字エンコーディング情報を伴う一連のバイトとしてインターネット経由で送信される。文字エンコーディング情報はHTTPレスポンスヘッダーまたはドキュメントのマークアップ内で特定される。ブラウザーはレンダリングするために、文字エンコーディング情報をバイトから文字列に変換する際に使用する。ブラウザーはページの文字情報を知ることなしに、正確にレンダリングできないため、多くのブラウザーは入力中に文字情報を検索しながら、JavaScriptの実行やページの描画前にいくつかのバイトをバッファリングします（注目すべき例外はInternet Explorer6, 7, 8）。

 ブラウザーによってバッファリングするバイト数は異なり、もし文字セットがなかった場合はデフォルトのエンコーディングを使用します。しかし、ひとたび必要なバイト量をバッファリングすればページのレンダリングを始めてしまうので、もし、デフォルトのエンコーディングと違う文字セットが指定されていたのなら再度パースをし、ページの再描画をしてしまう。時には、ミスマッチは外部リソースのURLにも影響するので、再リクエストをしなければならないかもしれない。

 これらの遅延を避けるために、あなた常に、文字エンコーディングをHTTPレスポンスヘッダーに明示しなければならない。文字エンコーディングはmeta http-equivタグでも指定可能だが、メタタグ指定では、Internet Explorer 8において*Lookahead Downloaderを無効*にする。先読みダウンローダーの無効化は実質的に、ページの読み込み時間を増加させてしまう。[マイクロソフトは](http://blogs.msdn.com/b/ieinternals/archive/2010/04/01/ie8-lookahead-downloader-fixed.aspx)、「私たちはWebデベロッパーに対して、HTTP Content-TypeレスポンスヘッダーでCHARSETの指定を強く推奨している。なぜならそうすることで、Lookahead Downloaderのメリットを保証するからだ」と述べていることに注意してもらいたい。

### Recommendations

__常にコンテンツタイプを指定する __  
ブラウザー文字セットをチェックする前に、最初に処理しているドキュメントのコンテンツタイプを決定しなければならない。もし、この指定がHTTPヘッダー、HTMLのメタタグで指定されていない場合は、ブラウザーはさまざまなアルゴリズムを使い推測しようとする。このプロセスは余計な遅延を引き起こし、セキュリティに脆弱性にもつながる可能性がある。パフォーマンスとセキュリティの面から、常に（text/htmlだけでなく）すべてのリソースに対してコンテンツタイプを指定する。

__正しい文字エンコーディングがセットされていることを確認する __  
HTTPヘッダーもしくはHTMLのメタタグで実際のドキュメントと同じ文字エンコーディングを指定することは重要だ。もしHTTPヘッダーとHTMLメタタグの両方で文字セットを指定するのなら、両方同じものかどうか確かめる。もし、ブラウザーが不正確または不一致なエンコーディングを検出した場合、ページの描画は不正確になり、再描画する遅延も発生する。詳細な文字セットに関する情報は、[HTML4.01仕様書](http://www.w3.org/TR/REC-html40/)の [Section 5.2, Character Encodings](http://www.w3.org/TR/REC-html40/charset.html#h-5.2) を参照すると良いだろう。

### Additional resources 

+ [Page Speed wiki](http://code.google.com/p/page-speed/wiki/SpecifyCharsetEarly)
+ [Browser Performance Issues with Charsets](http://zoompf.com/2009/12/browser-performance-issues-with-charsets)
+ [Performance Implications of "charset"](http://www.kylescholz.com/blog/2010/01/performance_implications_of_charset.html)