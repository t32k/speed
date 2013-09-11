## パフォーマンス改善のためのHTML5

原文：https://developers.google.com/speed/articles/html5-performance  
著者：Jens Meiert, Googleウェブマスター

推奨とされる経験：基本的なHTMLの知識

In HTML, you can already reduce content size significantly by omitting optional tags. HTML 5, which is still under development, offers you a couple more options to decrease your file size beside leaving out optional stuff. Here we’ll feature some basic measures to reduce content size a bit more, plus the async and defer attributes useful to improve script execution.

HTMLにおいて、省略可能なタグを削除することでコンテンツサイズを削減することが可能です。HTML5はまだ勧告されてはいませんが、

### DTD

HTML 5 allows you to set the document type by just stating (in any form, as the DTD is case-insensitive)

```html
<!DOCTYPE html>
```
上記のコードは以下のコードと比べて、

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
```

より短くDTDを記述することが理解できます。

### Encoding

When you specify the encoding of your document, HTML 5 is a bit easier to use and also more lightweight:

```html
<meta charset="utf-8">
```

will do what

```html
<meta http-equiv="content-type" content="text/html; charset=utf-8">
```

does otherwise; your browser usually knows that it’s dealing with HTML.

### `type` attributes

Another thing that works in HTML 5 – today – is that you can leave out all the type attributes related to MIME types. These are the attributes such as type="text/css" or type="text/javascript", which describe the type of contents the element in question contains or links to.

Instead of `<style type="text/css">` you can just write `<style>`; instead of writing `<script type="text/javascript">`, you can just use `<script>`. You can omit type="text/css" in almost all document types actually, even in XHTML, but HTML 5 makes all this even simpler since consistent.

### `async` (と `defer`)

async and defer are attributes to be used in conjunction with the script element.

To explain why these attributes are useful to improve performance, it’s best to see what happens when they’re not used – respective script would be fetched and executed immediately, before the user agent (browser) continues parsing the page. Sometimes this behavior is desirable, sometimes it’s not.

The new async attribute allows that respective script will be executed asynchronously, as soon as it is available. HTML 4, which already includes the defer attribute, states that defer “provides a hint to the user agent that the script is not going to generate any document content (e.g., no document.write in JavaScript) and thus, the user agent can continue parsing and rendering”.

If the async attribute is not used but the defer attribute is present, the script is executed when the page has finished parsing. The defer attribute may be specified even if the async attribute is specified. This allows legacy browsers that only know defer to fall back to the defer behavior instead of the synchronous blocking behavior that is the default.