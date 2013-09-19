## gzip圧縮のしくみ

原文：https://developers.google.com/speed/articles/gzip  
著者：Kevin Khaw & Eric Higgins, Google ウェブマスター  
推奨とされる経験：HTTPとHTMLに関する知識


[![How gzip works](http://img.youtube.com/vi/Mjab_aZsdxw/0.jpg)](http://www.youtube.com/watch?v=Mjab_aZsdxw)


### gzip圧縮の有無によるブラウザーリクエストの流れ

#### gzip圧縮無しの場合：

![](https://raw.github.com/t32k/speed/master/images/gzip_off.png)

__ブラウザー:__

+  サーバーに接続し、ページをリクエストする
+ `"Accept-Encoding: gzip"` でブラウザーがgzipをサポートしていることを通知

__サーバー:__

+ gzipをサポートしていないのでgzipリクエストを無視。未圧縮のページ送信

__ブラウザー:__

+ ページを受信
+ ページを表示


#### gzip圧縮有りの場合：

![](https://raw.github.com/t32k/speed/master/images/gzip_on.png)

__ブラウザー:__

+  サーバーに接続
+ `"Accept-Encoding: gzip"` でブラウザーがgzipをサポートしていることを通知

__サーバー:__

+ gzipサポートを確認
+ `"Content-Encoding: gzip"` ヘッダーを添えてgzip圧縮したページを送信


__ブラウザー:__

+ ページを受信
+ `"Content-Encoding: gzip"`にもとづいて、ページを解凍
+ ページを表示

### どのようにgzip圧縮されるのか

簡単に言えば、gzip圧縮はテキストファイル内で似たような文字列を見つけ一時的にそれらの文字列を置換することによって、全体のファイルサイズを縮小させます。 HTML/CSSファイルは通常、空白・タグ・またスタイル定義といった似たような文字列の繰り返しをたくさん含んでいるため、Webの世界でこそ、gzip圧縮すべきと言えるでしょう。

#### 例

この例は、違うタグを使用するよりも同一タグを使用することでコードのスニペットをより圧縮できることを説明します。

最初のスニペットは、ヘッドラインのタグを順番に使用しています。

未圧縮: 69 bytes  
圧縮: 85 bytes

```html
<h1>One</h1>
<h2>Two</h2>
<h3>Three</h3>
<h4>Four</h4>
<h5>Five</h5>
```

代わりに同じdivタグに変更すれば、ファイルサイズが10バイト増えてしまいますが、圧縮すれば前述のスニペットより66バイトまで縮小できます。

未圧縮: 79 bytes  
圧縮: 66 bytes

```html
<div>One</div>
<div>Two</div>
<div>Three</div>
<div>Four</div>
<div>Five</div>
```

divとspanタグだけ使うようにすれば、さらに圧縮率を高めるのかもしれないと多くの開発者は考えるかもしれません。それは確かにそうかもしれませんが、意味的に正しいマークアップをすることはアクセシビリティのために重要ですし、ページの可読性のためにも重要であることを忘れてはなりません。しかしながら、このような方法で最適化できる部分は存在するかと思います、そしてそれを適切に使用するかしないかは、あなた、開発者次第です。


### その他のリソース

+ [圧縮を有効にする - Page Speed](/docs/best-practices/payload.md#圧縮を有効にする)
