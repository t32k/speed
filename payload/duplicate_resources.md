---
layout: default
category: payload
date: 2012-10-01
title: 重複するリソースは1つのURLから提供する
---
## 重複するリソースは1つのURLから提供する

原文：[https://developers.google.com/speed/docs/best-practices/payload#duplicate_resources](https://developers.google.com/speed/docs/best-practices/payload#duplicate_resources)

### Overview

1つのURLでリソースを提供することは、重複ダウンロードと余計なRTTを削減するために重要なことだ。

### Details

時折、ページ上の複数の場所で同じリソースを参照する必要性が出てくる。画像は典型的な例だ。さらには.cssと.jsファイルはサイト上の複数のページで同じものを共有される可能性が高い。ページ上で同じリソースを含める必要がないのであれば、リソースは常に一貫したURLで提供されるべきだ。ひとつのリソースが常にひとつのURLを割り与えられることはいくつかのメリットがある。ブラウザーは余分に同じバイトのコピーをダウンロードせずに済むので、総ダウンロード量を削減できる。また多くのブラウザーは一回のセッション中でシングルURLのHTTPリクエストはキャッシュ可能であろうとなかろうと一回しか発行しないので、余分なラウンドトリップタイムも防ぐことができる。同じリソースが違うホスト名から提供されていないことを確認するのは、不要なDNSルックアップを避けるために特に重要だ。 絶対URLのホスト名がそのドキュメントを含んでいるホスト名かどうか、つまり相対URLと絶対URLが一致していることを確認する。例えば、もし、メインのページがwww.example.comで/images/example.gifと、www.example.com/images/example.gifのリソースを参照している場合、URLは一致している。しかしながら、そのページで、/images/example.gifと、mysite.example.com/images/example.gifを参照している場合、これは一致していない。

### Recommendations  

__サイト内のすべてのページで一貫したURLで共有リソースを提供する__  
複数のページで共有されているリソースが同一のURLで参照されていることを確認する。もし、それらのリソースが異なるドメインまたはホスト名で提供されている場合、それらのリソースは異なるホスト名が再提供されるよりも1つのホスト名から提供されるのが良いだろう。この場合、DNSルックアップのオーバーヘッドよりもキャッシュのメリットのほうが上回るだろう。例えば、もしmysite.example.comとyoursite.example.comで同じJSファイルを使用していて、mysite.example.comからyoursite.example.comにリンクが貼られていた場合（ともかくDNSルックアップは必須だろう）、mysite.example.comからのみJSファイルを提供したほうが理にかなっている。この方法の場合、ユーザーがyoursite.example.comを訪問した際には、JSファイルのブラウザーキャッシュが効いている可能性が高い。