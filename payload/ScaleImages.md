---
layout: default
category: payload
date: 2012-10-01
title: 変更後のサイズで画像を提供する
---
## 変更後のサイズで画像を提供する

原文：[https://developers.google.com/speed/docs/best-practices/payload#ScaleImages](https://developers.google.com/speed/docs/best-practices/payload#ScaleImages)

### Overview

適切なサイズの画像は多くのバイト数を削減することができる。

### Details

同じ画像を異なるサイズで表示したい場合、1つの画像をHTMLやCSSを使ってスケール変更するかもしれない。例えば、 250×250の大きな画像の10×10のサムネイルがあった場合、ユーザーに異なる2つのファイルをダウンロードさせるよりもマークアップでサムネイルのほうをリサイズして表示させたいと思うだろう。これはもっともな判断だ。なぜなら少なくとも大きい方の250×250は実際のサイズだからだ。しかしながら、もしマークアップで作成したインスタンスのすべてよりも大きなサイズの画像を提供した場合、不必要なバイトをネットワークを通じて送ってしまうことになる。画像編集ツールであなたのページで必要とする最も大きなサイズを画像を作成する。そして、マークアップの際に[画像のサイズも指定](http://localhost:4000/speed/rendering/SpecifyImageDimensions.html)していることを確認する。