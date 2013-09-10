---
layout: default
category: payload
date: 2012-10-01
title: 圧縮を有効にする
---
## 圧縮を有効にする

原文：[https://developers.google.com/speed/docs/best-practices/payload#GzipCompression](https://developers.google.com/speed/docs/best-practices/payload#GzipCompression)

### Overview

`gzip`や`deflate`でリソースを圧縮させることはネットワーク経由で送信するバイト数削減につながる。

### Details

ほとんどのモダンなブラウザーはHTML,CSS/JavaScriptファイルのデータ圧縮をサポートしている。これはコンテンツをネットワーク経由で送信する際にさらにコンパクトにでき、劇的にダウンロード時間を縮めることができる。

多くのWebサーバーは、サードパーティなモジュールまたはビルトインな機能を使い、ファイル送信する前にgzipフォーマットで圧縮させることが可能だ。圧縮を有効にするために、すべての圧縮可能なリソースに対して`Content-Encoding`ヘッダーに`gzip`フォーマットを設定する必要がある。同じ圧縮アルゴリズムのdeflateも使用できるが、このフォーマットは広く対応していないので、私たちはgzipを推奨する。圧縮処理がサーバーに負荷をかけている場合、事前に圧縮したファイルをキャッシュさせるといった設定も可能だ。  

注意すべきことはgzip化は大きなリソースにしか効果がない。圧縮・解凍のレイテンシーとオーバーヘッドを考慮すれば、gzip化するファイルはある程度のしきい値が存在する。私たちは150 - 1000バイトの間を最小限のレンジとして奨めており、150KB以下のファイルをgzip化すれば逆にファイルサイズを大きくしてしまう。

### Recommendations

__圧縮効率を最大限にするようにページコンテンツを記述する __  
圧縮を効率良くするために、以下のことを注意する

+ HTML/CSSコードを一貫性のあるものにする：
	+ 可能な限り、CSSのプロパティと値は同じ順序にする（例:アルファベット順）
	+ HTML属性も同じ順序で記述する（例：アルファベット順）hrefは最も一般的なので最初に記述し残りはアルファベット順にする。例えば、Googleの検索結果ページにおいてHTML属性をアルファベット順に記述した場合、gzip後のサイズが1.5%削減できた。
	+ 大文字・小文字を統一する（例：できる限り小文字に統一する）
	+ HTML属性の引用符を統一する（例：常にシングル、ダブル、もしくは使わないといった選択にする）
+ [JavaScript](http://localhost:4000/speed/payload/MinifyJS.html)/[CSS](/speed/payload/MinifyCSS.html)を縮小化する。JavaScript/CSSの縮小化は、外部ファイル、HTML内に書かれたインラインコード両方で圧縮性を高める。 


__画像やバイナリファイルに対してgzipしない__  
Webでサポートされている画像形式、同様にビデオ、PDFのようなバイナリファイルはすでに圧縮されているので、gzip化しても何のメリットもないどころか逆にサイズを大きくしてしまう。画像の最適化に関しては[こちらを参照](/speed/payload/CompressImages.html)。

### Additional resources

コンテンツの圧縮についてさらなる情報は [Use compression to make the web faster](http://code.google.com/speed/articles/use-compression.html) を参照。