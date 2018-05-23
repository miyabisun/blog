---
title: 【JavaScript】即時実行関数とは
date: 2013-03-12 21:14:46
tags:
  - JavaScript
id: javascript-tips
---

## 即時関数とは

JavaScriptにおいて、
下のような書き方を即時関数と呼ぶらしい。

```JavaScript
(function(){ 処理 })();
```

この書き方してるライブラリは山のようにあるけど、
名前もわからん動作もわからん、そもそも何だこの気持ち悪い書き方は？
紹介してる書籍やサイトにも辿り付けないで苦労した。

<!-- more -->

## 挙動
関数の尻に()を付けるとその場で実行される。

```JavaScript
function hoge() {
    // 処理
}
hoge(); // 関数hogeが実行される
```

しかし、{}は開始と終了時に行が変わると解釈される。

```
// その場で関数を実行したい
function hoge() {
    // 処理
}();

//　↓内部ではこのように解釈されるので関数hogeは動作しない＞＜；
function hoge() {
    // 処理
}
();
```

…で、`function(){}`を式として認識させれば挙動を邪魔されることはない
（下記参考サイトを参照）

```JavaScript
// 以下の書き方は全て同様
+function hoge(){ alert('hogeを実行') }()
-function hoge(){ alert('hogeを実行') }()
(function hoge(){ alert('hogeを実行') })()
```

しかし、+や-等を使うと戻り値が数値にキャストされてしまうので、
即時関数の書き方は下が一般的となる。

```
(function(){ 処理 })();
```

## メリット

グローバル変数が汚れない

```JavaScript
(function hoge(){ 処理 })();
alert(hoge); // underfindと表示される
```

## 参考サイト
三等兵「知ってて当然？初級者のためのJavaScriptで使う即時関数(function(){...})()の全て」
<http://d.hatena.ne.jp/sandai/20110824/p1>

泥のように「即時関数(function(){ ... })()の別の書き方いろいろ」
<http://blog.tojiru.net/article/197270788.html>

### コメント
クロージャー絡みで発見した。
クロージャーも勉強しなければ…

