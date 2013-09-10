---
layout: default
category: rendering
date: 2012-10-01
title: 画像のサイズを指定する
---
## 画像のサイズを指定する

原文：[https://developers.google.com/speed/docs/best-practices/rendering#SpecifyImageDimensions](https://developers.google.com/speed/docs/best-practices/rendering#SpecifyImageDimensions)

### Overview

すべての画像にwidthとheightを指定することは、不必要なリフロー・リペイントを発生させないためレンダリングが速くなる。

### Details

ブラウザーがページレイアウトをするとき、画像のような置換要素の周りではフローが発生する。それは非置換要素のサイズがわかれば、画像がダウンロードされる前であってもレンダリングを開始することができる。もしドキュメント内でサイズが指定されていない、実際の画像サイズと異なる場合、ブラウザーは画像をダウンロードした後にリフロー・リペイントを行わければならない。リフローを防ぐために、 HTMLの`<img>`タグまたはCSSですべての画像にwidthとheightを明示する。


### Recommendations

__実際の画像サイズにマッチしたサイズを指定する __  
動的にサイズを変更したwidth/heightを指定すべきではない。実際の画像ファイルが60×60ピクセルならば、30×30サイズのようなサイズをHTMLまたはCSSで指定しない。もし画像を小さくする必要があるのであれば画像編集ツールで変更し、実際の画像とマッチしたサイズを指定する（詳細は[画像を最適化する](http://localhost:4000/speed/payload/CompressImages.html)を参照）。


__必ず画像要素または親のブロックレベル要素にサイズを指定する __  
必ず`<img>`要素自身または親のブロックレベル要素にサイズを指定する。もし、親要素がブロック要素でなければ、サイズは無視される。また直接の親ではない祖先要素に対してはサイズは指定しない。