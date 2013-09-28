# ブラウザレンダリングの最適化

Once resources have been downloaded to the client, the browser still needs to load, interpret, and render HTML, CSS, and JavaScript code. By simply formatting your code and pages in ways that exploit the characteristics of current browsers, you can enhance performance on the client side.

+ Use efficient CSS selectors
+ Avoid CSS expressions
+ [CSSをドキュメントヘッドに含める](#CSSをドキュメントヘッドに含める)
+ [画像のサイズを指定する](#画像のサイズを指定する)
+ [文字セットを指定する](#文字セットを指定する)


## Use efficient CSS selectors

### 概要

Avoiding inefficient key selectors that match large numbers of elements can speed up page rendering.

### 詳細

As the browser parses HTML, it constructs an internal document tree representing all the elements to be displayed. It then matches elements to styles specified in various stylesheets, according to the standard CSS cascade, inheritance, and ordering rules. In Mozilla's implementation (and probably others as well), for each element, the CSS engine searches through style rules to find a match. The engine evaluates each rule from right to left, starting from the rightmost selector (called the "key") and moving through each selector until it finds a match or discards the rule. (The "selector" is the document element to which the rule should apply.)

According to this system, the fewer rules the engine has to evaluate the better. So, of course, removing unused CSS is an important step in improving rendering performance. After that, for pages that contain large numbers of elements and/or large numbers of CSS rules, optimizing the definitions of the rules themselves can enhance performance as well. The key to optimizing rules lies in defining rules that are as specific as possible and that avoid unnecessary redundancy, to allow the style engine to quickly find matches without spending time evaluating rules that don't apply.

The following categories of rules are considered to be inefficient:

__Rules with descendant selectors__  
For example:  
__Rules with the universal selector as the key__

```css
body * {...}
.hide-scrollbars * {...}
```

__Rules with a tag selector as the key__

```css
ul li a {...}
#footer h3 {...}
* html #atticPromo ul li a {...]
```

Descendant selectors are inefficient because, for each element that matches the key, the browser must also traverse up the DOM tree, evaluating every ancestor element until it finds a match or reaches the root element. The less specific the key, the greater the number of nodes that need to be evaluated.

__Rules with child or adjacent selectors__  
For example:  
__Rules with the universal selector as the key__

```css
body > * {...}
.hide-scrollbars > * {...}
```

__Rules with a tag selector as the key__

```css
ul > li > a {...}
#footer > h3 {...}
```

Child and adjacent selectors are inefficient because, for each matching element, the browser has to evaluate another node. It becomes doubly expensive for each child selector in the rule. Again, the less specific the key, the greater the number of nodes that need to be evaluated. However, while inefficient, they are still preferable to descendant selectors in terms of performance.


__Rules with overly qualified selectors__  
For example:

```css
ul#top_blue_nav {...}
form#UserLogin {...}
```

ID selectors are unique by definition. Including tag or class qualifiers just adds redundant information that needs to be evaluated needlessly.

__Rules that apply the :hover pseudo-selector to non-link elements__  
For example:

```css
h3:hover {...}
.foo:hover {...}
#foo:hover {...}
div.faa :hover {...}
```

The :hover pseudo-selector on non-anchor elements is known to make IE7 and IE8 slow in some cases*.  When a strict doctype is not used, IE7 and IE8 will ignore :hover on any element other than anchors. When a strict doctype is used, :hover on non-anchors may cause performance degradation. 

* See a bug report at http://connect.microsoft.com/IE/feedback/ViewFeedback.aspx?FeedbackID=391387.

### 推奨

__Avoid a universal key selector.__  
Allow elements to inherit from ancestors, or use a class to apply a style to multiple elements.

__Make your rules as specific as possible.__  
Prefer class and ID selectors over tag selectors.

__Remove redundant qualifiers.__
These qualifiers are redundant:

+ ID selectors qualified by class and/or tag selectors
+ Class selectors qualified by tag selectors (when a class is only used for one tag, which is a good design practice anyway).

__Avoid using descendant selectors, especially those that specify redundant ancestors.__  
For example, the rule body ul li a {...} specifies a redundant body selector, since all elements are descendants of the body tag.

__Use class selectors instead of descendant selectors.__  
For example, if you need two different styles for an ordered list item and an ordered list item, instead of using two rules:

```css
ul li {color: blue;}
ol li {color: red;}
```

You could encode the styles into two class names and use those in your rules; e.g:

```css
.unordered-list-item {color: blue;}
.ordered-list-item {color: red;}
```

If you must use descendant selectors, prefer child selectors, which at least only require evaluation of one additional node, not all the intermediate nodes up to an ancestor.

__Avoid the :hover pseudo-selector for non-link elements for IE clients.__  
If you use :hover on non-anchor elements, test the page in IE7 and IE8 to be sure your page is usable.   If you find that :hover is causing performance issues, consider conditionally using a JavaScript onmouseover event handler for IE clients.

### 追加のリソース

+ For more details on efficient CSS rules with Mozilla, see Writing Efficient CSS for Use in the Mozilla UI.
+ For complete information on CSS, see the Cascading Style Sheets Level 2 Revision 1 (CSS 2.1) Specification. For information on CSS selectors specifically, see Chapter 5.


## Avoid CSS expressions

### Overview

CSS expressions degrade rendering performance; replacing them with alternatives will improve browser rendering for IE users.

__Note:__ This best practices in this section apply only to Internet Explorer 5 through 7, which support CSS expressions. CSS expressions are deprecated in Internet Explorer 8, and not supported by other browsers.


### Details

Internet Explorer 5 introduced CSS expressions, or "dynamic properties", as a means of dynamically changing document properties in response to various events. They consist of JavaScript expressions embedded as the values of CSS properties in CSS declarations. For the most part, they are used for the following purposes:

+ To emulate standard CSS properties supported by other browsers but not yet implemented by IE.
+ To provide dynamic styling and advanced event handling in a more compact and convenient way than writing full-blown JavaScript-injected styles.

Unfortunately, the performance penalty imposed by CSS expressions is considerable, as the browser reevaluates each expression whenever any event is triggered, such as a window resize, a mouse movement and so on. The poor performance of CSS expressions is one of the reasons they are now deprecated in IE 8. If you have used CSS expressions in your pages, you should make every effort to remove them and use other methods to achieve the same functionality.

### Recommendations

__Use standard CSS properties if possible.__   
IE 8 is fully CSS-standards-compliant; it supports CSS expressions only if run in "compatibility" mode, but it does not support them in "standards" mode. If you do not need to maintain backwards compatibility with older versions of IE, you should convert any instances of expressions used in place of standard CSS properties to their CSS standard counterparts. For a complete list of CSS properties and IE versions that support them, see the MSDN CSS Attributes Index. If you do need to support older versions of IE in which the desired CSS properties are not available, use JavaScript to achieve the equivalent functionality.

__Use JavaScript to script styles.__  
If you are using CSS expressions for dynamic styling, it makes sense to rewrite them as pure JavaScript to both improve performance in IE and get the benefit of supporting the same functionality in other browsers at the same time. In this example given on the MSDN page on Dynamic Properties, a CSS expression is used to center an HTML block whose dimensions can change at runtime, and to re-center that block every time the window is resized:


```html
<div id="oDiv" style="background-color: #CFCFCF; position: absolute;
left:expression(document.body.clientWidth/2-oDiv.offsetWidth/2); 
 top:expression(document.body.clientHeight/2-oDiv.offsetHeight/2)">Example DIV</div>
```

Here's an equivalent example using JavaScript and standard CSS:

```html
<style>
  #oDiv { position: absolute; background-color: #CFCFCF;}
</style>

<script type="text/javascript">
 // Check for browser support of event handling capability
  if (window.addEventListener) {
  window.addEventListener("load", centerDiv, false);
 window.addEventListener("resize", centerDiv, false);
  } else if (window.attachEvent) {
  window.attachEvent("onload", centerDiv);
  window.attachEvent("onresize", centerDiv);
  } else {
  window.onload = centerDiv;
  window.resize = centerDiv;
  }
	 
  function centerDiv() {
  var myDiv = document.getElementById("oDiv");
  var myBody = document.body;
  var bodyWidth = myBody.offsetWidth;
 
  //Needed for Firefox, which doesn't support offsetHeight
  var bodyHeight;
 if (myBody.scrollHeight) 
 bodyHeight = myBody.scrollHeight;
 else bodyHeight = myBody.offsetHeight;
 
  var divWidth = myDiv.offsetWidth;
 
  if (myDiv.scrollHeight)
   var divHeight = myDiv.scrollHeight;
   else var divHeight = myDiv.offsetHeight;
 
 myDiv.style.top = (bodyHeight - divHeight) / 2;
  myDiv.style.left = (bodyWidth - divWidth) / 2; 
  }

</script>
```

If you are using CSS expressions to emulate CSS properties that aren't available in earlier versions of IE, you should provide JavaScript code for those cases with a version test to disable it for browsers that do support CSS. For example, the max-width property, which forces text to wrap around at a certain number of pixels, was not supported until IE 7. As a workaround, this CSS expression provides that functionality for IE 5 and 6:

```html
p { width: expression( document.body.clientWidth > 600 ? "600px" : "auto" ); }
```
To replace the CSS expression with equivalent JavaScript for the IE versions that don't support this property, you could use something like the following:

```html
<style>
  p { max-width: 300px; }
</style>

<script type="text/javascript">

  if ((navigator.appName == "Microsoft Internet Explorer") && (parseInt(navigator.appVersion) < 7))
  window.attachEvent("onresize", setMaxWidth);
	
  function setMaxWidth() {
  var paragraphs = document.getElementsByTagName("p");
  for ( var i = 0; i < paragraphs.length; i++ ) 
  paragraphs[i].style.width = ( document.body.clientWidth > 300 ? "300px" : "auto" );

</script>
```

## CSSをドキュメントヘッドに含める

### 概要

インラインスタイルと`<link>`はbodyからheadへ移動させるとレンダリングが改善される。

### 詳細

インラインスタイルと外部CSSをHTMLのbodyに記述することはレンダリングパフォーマンスに悪影響を与える。ブラウザーはすべての外部CSSが読み込まれるまでページのレンダリングをブロックする。
`<style>`タグで記述されたインラインスタイルはリフローを発生させるので、インラインスタイルも同様にページのheadで外部CSSを参照することが重要だ。スタイルシートが一番最初にダウンロード・パースされることによって、ブラウザーのプログレッシブレンダリングが可能となる。

### 推奨

+ HTML4.01仕様書（[section12.3](http://www.w3.org/TR/html4/struct/links.html#h-12.3)）によると、外部CSSは常に `<head>` 内に `<link>` タグで記述すると書かれており、`@import`は使用しない。またスクリプトとスタイルシートを正しい順序で記述する。
+ `<style>` タグも `<head>` 部分に記述する。



## 画像のサイズを指定する

### 概要

すべての画像にwidthとheightを指定することは、不必要なリフロー・リペイントを発生させないためレンダリングが速くなる。

### 詳細

ブラウザーがページレイアウトをするとき、画像のような置換要素の周りではフローが発生する。それは非置換要素のサイズがわかれば、画像がダウンロードされる前であってもレンダリングを開始することができる。もしドキュメント内でサイズが指定されていない、実際の画像サイズと異なる場合、ブラウザーは画像をダウンロードした後にリフロー・リペイントを行わければならない。リフローを防ぐために、 HTMLの`<img>`タグまたはCSSですべての画像にwidthとheightを明示する。

### 推奨

__実際の画像サイズにマッチしたサイズを指定する __  
動的にサイズを変更したwidth/heightを指定すべきではない。実際の画像ファイルが60×60ピクセルならば、30×30サイズのようなサイズをHTMLまたはCSSで指定しない。もし画像を小さくする必要があるのであれば画像編集ツールで変更し、実際の画像とマッチしたサイズを指定する（詳細は[画像を最適化する](http://localhost:4000/speed/payload/CompressImages.html)を参照）。


__必ず画像要素または親のブロックレベル要素にサイズを指定する __  
必ず`<img>`要素自身または親のブロックレベル要素にサイズを指定する。もし、親要素がブロック要素でなければ、サイズは無視される。また直接の親ではない祖先要素に対してはサイズは指定しない。



## 文字セットを指定する

### 概要

HTTPレスポンスヘッダーで文字セットを明示することは、HTMLパース・スクリプト実行を早めてくれる。

### 詳細

HTMLドキュメントは、文字エンコーディング情報を伴う一連のバイトとしてインターネット経由で送信される。文字エンコーディング情報はHTTPレスポンスヘッダーまたはドキュメントのマークアップ内で特定される。ブラウザーはレンダリングするために、文字エンコーディング情報をバイトから文字列に変換する際に使用する。ブラウザーはページの文字情報を知ることなしに、正確にレンダリングできないため、多くのブラウザーは入力中に文字情報を検索しながら、JavaScriptの実行やページの描画前にいくつかのバイトをバッファリングします（注目すべき例外はInternet Explorer6, 7, 8）。

 ブラウザーによってバッファリングするバイト数は異なり、もし文字セットがなかった場合はデフォルトのエンコーディングを使用します。しかし、ひとたび必要なバイト量をバッファリングすればページのレンダリングを始めてしまうので、もし、デフォルトのエンコーディングと違う文字セットが指定されていたのなら再度パースをし、ページの再描画をしてしまう。時には、ミスマッチは外部リソースのURLにも影響するので、再リクエストをしなければならないかもしれない。

 これらの遅延を避けるために、あなた常に、文字エンコーディングをHTTPレスポンスヘッダーに明示しなければならない。文字エンコーディングはmeta http-equivタグでも指定可能だが、メタタグ指定では、Internet Explorer 8において*Lookahead Downloaderを無効*にする。先読みダウンローダーの無効化は実質的に、ページの読み込み時間を増加させてしまう。[マイクロソフトは](http://blogs.msdn.com/b/ieinternals/archive/2010/04/01/ie8-lookahead-downloader-fixed.aspx)、「私たちはWebデベロッパーに対して、HTTP Content-TypeレスポンスヘッダーでCHARSETの指定を強く推奨している。なぜならそうすることで、Lookahead Downloaderのメリットを保証するからだ」と述べていることに注意してもらいたい。

### 推奨

__常にコンテンツタイプを指定する __  
ブラウザー文字セットをチェックする前に、最初に処理しているドキュメントのコンテンツタイプを決定しなければならない。もし、この指定がHTTPヘッダー、HTMLのメタタグで指定されていない場合は、ブラウザーはさまざまなアルゴリズムを使い推測しようとする。このプロセスは余計な遅延を引き起こし、セキュリティに脆弱性にもつながる可能性がある。パフォーマンスとセキュリティの面から、常に（text/htmlだけでなく）すべてのリソースに対してコンテンツタイプを指定する。

__正しい文字エンコーディングがセットされていることを確認する __  
HTTPヘッダーもしくはHTMLのメタタグで実際のドキュメントと同じ文字エンコーディングを指定することは重要だ。もしHTTPヘッダーとHTMLメタタグの両方で文字セットを指定するのなら、両方同じものかどうか確かめる。もし、ブラウザーが不正確または不一致なエンコーディングを検出した場合、ページの描画は不正確になり、再描画する遅延も発生する。詳細な文字セットに関する情報は、[HTML4.01仕様書](http://www.w3.org/TR/REC-html40/)の [Section 5.2, Character Encodings](http://www.w3.org/TR/REC-html40/charset.html#h-5.2) を参照すると良いだろう。

### その他のリソース

+ [Page Speed wiki](http://code.google.com/p/page-speed/wiki/SpecifyCharsetEarly)
+ [Browser Performance Issues with Charsets](http://zoompf.com/2009/12/browser-performance-issues-with-charsets)
+ [Performance Implications of "charset"](http://www.kylescholz.com/blog/2010/01/performance_implications_of_charset.html)

---

原文：https://developers.google.com/speed/docs/best-practices/rendering
