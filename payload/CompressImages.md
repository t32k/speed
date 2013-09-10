---
layout: default
category: payload
date: 2012-10-01
title: 画像を最適化する
---
## 画像を最適化する

原文：[https://developers.google.com/speed/docs/best-practices/payload#CompressImages](https://developers.google.com/speed/docs/best-practices/payload#CompressImages)

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