---
title: 【Excel】セルの値に従って別シートのセルを読み込む
date: 2013-02-25 21:19:53
tags: Excel
id: excel-tips
---

## 目的
一覧表に各シートの件数やタイトルを記入する
同一セル内でも行う方法

## 方法1：INDIRECT

```
=INDIRECT($B5&"!F3")
```

INDIRECTに「シート名!A1」みたいな文字列を投げ込むと、
「=シート名!A1」として扱ってくれるというネタ

<!-- more -->

### 参考資料
「excel関数 別シート参照」でぐぐる

はてな？excelで別シートを参照するとき...
<http://q.hatena.ne.jp/1166860036>

別シートへのセル参照をオートフィルで−INDIRECT関数
<http://www.relief.jp/itnote/archives/001697.php>

## 方法2：OFFSET
```
=OFFSET(基準セル,行位置,列位置,行高さ,列幅)
=OFFSET(A1,2,0)
// こっちは2次元配列を書き換えてテスト項目表を作成するのに使った
```

### 参考資料
ＥＸＣＥＬでのセル座標を指定する数式について
<http://questionbox.jp.msn.com/qa2030149.html>

