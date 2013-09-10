
原文：https://developers.google.com/speed/docs/best-practices/payload#GzipCompression


## 圧縮を有効にする


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

## 重複するリソースは1つのURLから提供する

### Overview

1つのURLでリソースを提供することは、重複ダウンロードと余計なRTTを削減するために重要なことだ。

### Details

時折、ページ上の複数の場所で同じリソースを参照する必要性が出てくる。画像は典型的な例だ。さらには.cssと.jsファイルはサイト上の複数のページで同じものを共有される可能性が高い。ページ上で同じリソースを含める必要がないのであれば、リソースは常に一貫したURLで提供されるべきだ。ひとつのリソースが常にひとつのURLを割り与えられることはいくつかのメリットがある。ブラウザーは余分に同じバイトのコピーをダウンロードせずに済むので、総ダウンロード量を削減できる。また多くのブラウザーは一回のセッション中でシングルURLのHTTPリクエストはキャッシュ可能であろうとなかろうと一回しか発行しないので、余分なラウンドトリップタイムも防ぐことができる。同じリソースが違うホスト名から提供されていないことを確認するのは、不要なDNSルックアップを避けるために特に重要だ。 絶対URLのホスト名がそのドキュメントを含んでいるホスト名かどうか、つまり相対URLと絶対URLが一致していることを確認する。例えば、もし、メインのページがwww.example.comで/images/example.gifと、www.example.com/images/example.gifのリソースを参照している場合、URLは一致している。しかしながら、そのページで、/images/example.gifと、mysite.example.com/images/example.gifを参照している場合、これは一致していない。

### Recommendations  

__サイト内のすべてのページで一貫したURLで共有リソースを提供する__  
複数のページで共有されているリソースが同一のURLで参照されていることを確認する。もし、それらのリソースが異なるドメインまたはホスト名で提供されている場合、それらのリソースは異なるホスト名が再提供されるよりも1つのホスト名から提供されるのが良いだろう。この場合、DNSルックアップのオーバーヘッドよりもキャッシュのメリットのほうが上回るだろう。例えば、もしmysite.example.comとyoursite.example.comで同じJSファイルを使用していて、mysite.example.comからyoursite.example.comにリンクが貼られていた場合（ともかくDNSルックアップは必須だろう）、mysite.example.comからのみJSファイルを提供したほうが理にかなっている。この方法の場合、ユーザーがyoursite.example.comを訪問した際には、JSファイルのブラウザーキャッシュが効いている可能性が高い。


## CSS を縮小する

### Overview

CSSコードを縮小させることはコードのバイト数を削減させることにつながり、ダウンロード・パース・実行時間を速めることができる。

### Details

CSSの縮小化は、JavaScriptの縮小化と同様のメリットがある。ネットワークのレイテンシーを削減したり圧縮効率を高めたり、ブラウザーの読み込みと実行時間を速めてくれる。

例えば、[YUI Compressor](http://developer.yahoo.com/yui/compressor/) や [cssmin.js](http://www.phpied.com/cssmin-js/) のような、CSSを縮小化する無料のツールがいくつかある。

Tip: CSSファイルに対してPage Speedを起動させると自動的にcssmin.jsが起動し、縮小化されたファイルが [configurable directory](https://developers.google.com/speed/docs/insights/using_firefox#savefiles) に保存される。


## HTML を縮小する

### Overview

そのページに含まれるインラインのJS/CSSを含めてHTMLを圧縮させることはコードのバイト数を削減させることにつながり、ダウンロード・パース・実行時間を速めることができる。

### Details

HTMLの縮小化は、JS/CSSの縮小化と同様のメリットがあります。ネットワークのレイテンシーを削減したり圧縮率を高めたり、ブラウザーの読み込みと実行時間を速めてくれる。さらにHTMLには、しばしばインラインのJS`<script>`とCSS`<style>`も含まれているため、それらにも同様に圧縮のメリットがあります。

> *Note:* このルールは実験的であり、現在のバージョンでは厳格なHTMLの整合性というよりもサイズ減量にフォーカスしている。将来のバージョンでもこのような整合性もルールに追加する予定だ。現在の詳しい挙動に関しては [Page Speed wiki](http://code.google.com/p/page-speed/wiki/MinifyHtml) を参照。

*Tip:* HTMLファイルに対してPage Speedを起動させると自動的にHTML compactor（インラインのJavaScriptとCSSにはJSMinとcssmin.jsがそれぞれ適用）が起動し、縮小化されたファイルが [configurable directory](https://developers.google.com/speed/docs/insights/using_firefox#savefiles) に保存されます。


## JavaScript を縮小する

### Overview

JavaScriptコードを圧縮させることはコードのバイト数を削減させることにつながり、ダウンロード・パース・実行時間を速めることが可能。

### Details

コードを縮小化するということは、余分なスペース、改行、インデントのような不要なバイトを取り除くということだ。JavaScriptをコンパクトに保つ利点はいくつかあり、まず最初にキャッシュさせたくないインライン・外部JSがある場合、ファイルサイズが小さければページのダウンロード毎に発生するレイテンシーを抑えることができる。2つ目に、縮小化は外部JS、HTML内に記述されたインラインJSの[圧縮](/speed/payload/GzipCompression.html)率をさらに高める。最後に、より小さなファイルは読み込みと起動が素早くなる。

[Closure Compiler](https://developers.google.com/closure/compiler/) や [JSMin](http://www.crockford.com/javascript/jsmin.html),  [YUI Compressor](http://developer.yahoo.com/yui/compressor/) などのいくつかのツールは無料でJavaScriptを縮小化できる。また、これらのツールを利用し縮小化・開発用ファイルのリネーム・プロダクションディレクトリに保存といったビルドプロセスを作ることも可能だ。4096バイト以上のJSファイルであれば縮小化することを推奨する。そして25バイト以上の削減（これ以下であればパフォーマンス対するメリットはない）が確認できるだろう。

Tip: JSファイルに対してPage Speedを起動させると自動的にClosure Complier(もし利用できるのであれば)か、JSMin（インラインJSに対して、Compilerが利用できない時）が起動し、縮小化されたファイルが [configurable directory](https://developers.google.com/speed/docs/insights/using_firefox#savefiles) に保存される。


## 変更後のサイズで画像を提供する


### Overview

適切なサイズの画像は多くのバイト数を削減することができる。

### Details

同じ画像を異なるサイズで表示したい場合、1つの画像をHTMLやCSSを使ってスケール変更するかもしれない。例えば、 250×250の大きな画像の10×10のサムネイルがあった場合、ユーザーに異なる2つのファイルをダウンロードさせるよりもマークアップでサムネイルのほうをリサイズして表示させたいと思うだろう。これはもっともな判断だ。なぜなら少なくとも大きい方の250×250は実際のサイズだからだ。しかしながら、もしマークアップで作成したインスタンスのすべてよりも大きなサイズの画像を提供した場合、不必要なバイトをネットワークを通じて送ってしまうことになる。画像編集ツールであなたのページで必要とする最も大きなサイズを画像を作成する。そして、マークアップの際に[画像のサイズも指定](http://localhost:4000/speed/rendering/SpecifyImageDimensions.html)していることを確認する。


## 画像を最適化する

### Overview

適切なフォーマットと圧縮された画像はデータ削減につながる。

### Details

Fireworksのようなソフトで作成された画像は、目に見えない程度の減色をしても、キロバイトのコメントや必要以上のカラーが含まれている。最適化が不十分な画像は必要以上なスペースを含んでいるため、帯域幅の狭いユーザーにとっては、画像サイズを最小限にすることは非常に重要なことだ。

すべての画像に対してベーシックとアドバンスな最適化を行うべきだ。ベーシックな最適化とは、画像から不要なスペースを取り除いたり、画像のメタ情報を削除したり、適切なフォーマットで保存することだ。この最適化は [GIMP](http://www.gimp.org/) のような画像編集ツールがあれば誰にでもできる。アドバンスな最適化は、JPEGとPNGファイルに対して更にロスレス圧縮をかけることだ。この手法は25バイト以上の削減がなければパフォーマンス的に意味が無いので、どれくらい効果がでたのか確認すべきだ。

### Recommendations  

__適切な画像フォーマットを選択する __  
画像フォーマットはファイルサイズに大きく関係あるため、次のガイドラインを使用する：

+ PNGはたいていの場合常に、GIFよりも優れたフォーマットであり最適な選択と言える。IE 4.0b1+, Mac IE 5.0+, Opera 3.51+とNetscape 4.04+と、すべてのバージョンのSafariとFirefoxは透過も含めて十分に対応している。IEの4から6はアルファチャンネルの透過（半透明）をサポートしていないが、(GIFのようなブーリアン透過)1ビットの透過と256色以下のPNGはサポートしている。IE7,8はアルファ透明フィルターが適用されている要素を除いて、アルファ透過をサポートしている。GIMPのRGBモードよりもインデックスモードを使用すれば適切なPNGを生成・コンバートできる。もし、IE3.xレベルのブラウザーに対応する必要があるのならば、これらのブラウザーに対して代替GIFを提供する。
+ とても小さな、もしくはシンプルなグラフィックのアニメーションを含んだ画像はGIFフォーマットを選択する（例：10x10ピクセルの以下の画像や、カラーパレットが3色未満のもの）。もし、GIFのほうがよりベターだと考えるのなら、両方試してみて、サイズの小さい方を選ぶ。
+ 写真的な画像はJPEGフォーマットを選択する。

__画像コンプレッサーを使用する__  

JPEG, PNGファイルに対してさらにロスレス圧縮ができ、しかも画像クオリティを変化させないツールがいくつか利用できる。私たちはJPEGの場合は、[jpegtran](http://jpegclub.org/) もしくは [jpegoptim](http://freecode.com/projects/jpegoptim)（Linuxのみ対応; --strip-allオプションで起動）PNGの場合、[OptiPNG](http://optipng.sourceforge.net/) または [PNGOUT](http://www.advsys.net/ken/util/pngout.htm) を推奨する。

*Tip:* ページが参照しているJPEFとPNGに対してPage Speedを起動させると自動的に圧縮されたファイルが [configurable directory](https://developers.google.com/speed/docs/insights/using_firefox#savefiles) に保存される。