---
title: 【働クリッカー】これが最速だと思います。
date: 2016-02-20 13:53:12
tags:
  - 働クリッカー
  - JavaScript
  - アルゴリズム
id: working-clicker-answer
---

## 能書き

前回再帰関数で0.24sの関数を作成した。

### 前回の調査でわかってること

- start()関数を叩くまでタイムは動かない
- 以下の3変数は上書き出来ない(getterを使った関数)
  - cash
  - items
  - prices
- 用意されているloop関数は超絶遅い
- shikakuとprogramming以外は取る必要無し
  - 最速を狙う場合、1秒以内の極僅かな時間で何万work()関数を実行する為
- if文は出来れば減らしたい
- for, whileより再帰関数が若干速い？

<!-- more -->

### 作戦

ifもforもwhileも再帰関数も何も使わないのが一番速いに決まっている。
そしてstart()関数を叩くまでにはいくら時間を使っても計上されない。

だから作るんだよ…work()とpurchase()関数しか実行せずに1億円貯まる関数を…！
そしてstart()直後にその関数を叩けば最速に違いない。

## 調査

### 各種ワードチェック

```JavaScript
print(items);
// { affiliate: 0, kabu: 0, programming: 0, shikaku: 0, tochi: 0 }
```

### 使える関数確認

これが出来ないと相当キツイから確認
必須関数（下記を確認していく）

- eval
- Array.map
- Array.forEach
- Array.join

```JavaScript
print([1, 2, 3].map(function(it){return it + 1;}));
// [ 2, 3, 4 ]

[1, 2, 3].forEach(function(it){print(it * 2);});
// 2
// 4
// 6

eval("[1, 2, 3].forEach(function(it){print(it * 2);});");
// 2
// 4
// 6

print([1, 2, 3].join(""))
// 123
```

普通のJavaScriptと同じことが出来る、勝ち確

### shikakuと賃金の相関

下記のような調査専用プログラムを作って確認。
iの数を4,000程度まで増やして確認したが、
shikaku 1点につき、次の値段+12.5、賃金+1.5の割合で増加し続ける。

```JavaScript
works = function(num){
  return function(){
    for (var i = 0; i < num; i=(i+1)|0){
      work();
    }
  };
}

wages = function(){
  var now = cash;
  work();
  return cash - now;
}

data = function(target){
  return function(){
    print([items[target], prices[target], wages(), Math.round(wages() / prices[target] * 1000)/1000]);
  }
}

start();
for (var i = 0; i < 20; i=(i+1)|0) {
  data('shikaku')();
  works(10)();
  purchase('shikaku');
}

// [ 0, 10, 2, 0.2 ]
// [ 1, 22, 3.5, 0.182 ]
// [ 2, 35, 5, 0.143 ]
// [ 3, 47, 6.5, 0.149 ]
// [ 4, 60, 8, 0.133 ]
// [ 5, 72, 9.5, 0.139 ]
// [ 6, 85, 11, 0.129 ]
// [ 7, 97, 12.5, 0.134 ]
// [ 8, 110, 14, 0.127 ]
// [ 9, 122, 15.5, 0.131 ]
// [ 10, 135, 17, 0.126 ]
// [ 11, 147, 18.5, 0.129 ]
// [ 12, 160, 20, 0.125 ]
// [ 13, 172, 21.5, 0.128 ]
// [ 14, 185, 23, 0.124 ]
// [ 15, 197, 24.5, 0.127 ]
// [ 16, 210, 26, 0.124 ]
// [ 17, 222, 27.5, 0.126 ]
// [ 18, 235, 29, 0.123 ]
// [ 19, 247, 30.5, 0.126 ]
// [ 20, 260, 32, 0.123 ]
```

### shikakuと賃金の相関

購入品だけ変更した調査関数を実行。
programming 1点につき、次の値段+21,250、賃金+20の割合で増加し続ける。

なんてこった…値段に対して効果が全然見合ってないじゃないか…！
市場価値で言えばプログラミングスキルなんてゴミ同然だったのだ！
ITバブルの結果IT土方が大量に出てきたのも頷ける。

```JavaScript
start();
for (var i = 0; i < 20; i=(i+1)|0) {
  data('programming')();
  works(10000)();
  purchase('programming');
}

// [ 0, 17000, 2, 0 ]
// [ 1, 38250, 22, 0.001 ]
// [ 2, 59500, 42, 0.001 ]
// [ 3, 80750, 62, 0.001 ]
// [ 4, 102000, 82, 0.001 ]
// [ 5, 123250, 102, 0.001 ]
// [ 6, 144500, 122, 0.001 ]
// [ 7, 165750, 142, 0.001 ]
// [ 8, 187000, 162, 0.001 ]
// [ 9, 208250, 182, 0.001 ]
// [ 10, 229500, 202, 0.001 ]
// [ 11, 250750, 222, 0.001 ]
// [ 12, 272000, 242, 0.001 ]
// [ 13, 293250, 262, 0.001 ]
// [ 14, 314500, 282, 0.001 ]
// [ 15, 335750, 302, 0.001 ]
// [ 16, 357000, 322, 0.001 ]
// [ 17, 378250, 342, 0.001 ]
// [ 18, 399500, 362, 0.001 ]
// [ 19, 420750, 382, 0.001 ]
```

## アルゴリズムを考える

### 調査結果

- evalを始めとする必須関数は全て使用可能（やったね！）
- 初期賃金: 2円
- shikaku
  - 初期価格: 10円
  - 取得
    - 価格: +12.5円
    - 賃金: +1.5円
- programming
  - 初期価格: 17,000円
  - 取得
    - 価格: +21,250円
    - 賃金: +20円

調査するまでに段階ではprogrammingの方が効率が良さそうと思い込んでいたので、
shikakuを取り続けるフェイズ→programmingを取り続けるフェイズ→work()を連打
…という風に考えていたのだが、効率が悪すぎる為に別の方法を取る。

### 定義

これで全て判明したように思えるが、重要な点がまだブラックボックスだ。
それは**work(), purchase()関数のコスト**である。
いくらshikakuのパフォーマンスが良く見えようと、もしpurchase()関数のコストがとんでもない程重く、work()関数のコストが軽ければprogrammingを取得すべきだ。

今回は面倒なのでそこの調査はパスし、下記のように各関数を仮定、大体同じコストだろうという検討をつけた。
※関数定義部分は憶測です。

```JavaScript
var work, add_cash, arr_items, purchase;
work = function() {
  add_cash(2 + items.shikaku * 1.5 + items.programming * 20);
}
add_cash = function(add) {
  window.cash = window.cash + add;
  check_record(); // 実績を達成したか否かのメソッドを叩く
}
arr_items = ['shikaku', 'affiliate', 'kabu', 'tochi', 'programming'];
purchase = function(target) {
  var str_target = (typeof(target) == 'number' ? arr_items[target] : target);
  if (cash >= prices[str_target]) {
    window.cash = window.cash - prices[str_target];
    window.items[prices[str_target]] = window.items[prices[str_target]] + 1;
    check_record(); // 実績を達成したか否かのメソッドを叩く
  }
}
```

概ねこんな感じだろう。
重いのはif文であれこれの値を見まくる必要のある実績確認処理なのだ。
(全く同じcheck_record()関数を使っているとは考えづらいが…)
一旦は**work(), purchase()関数のコストはほぼ同等**と仮定する。

### 基本ロジック

shikakuもprogrammingも取らずにwork()だけ実行しまくったら、1億円/2円=5千万回のwork()を叩く必要がある。

ここで、もしshikaku(1個目)を取るとどうなるだろう？
shikaku(1回目)の金額は10円なので、取るのにwork()5回、purchase()1回の6コストを支払う必要がある。
その代わり1億円稼ぐ為のwork()実行回数が減少するというメリットが得られる。
計算すると「5work()+1purchase()+(1億円/3.5円=28,571,429work())」のコストとなり、いきなり半分近くのコストとなる事が分かる。

これを何度も繰り返してshikakuを取ってもコストが下げられないと判明するまでのwork()とpurchase()の組み合わせを求める。

programmingとの関係も一応確認しておく必要がある。
いくら効率が悪いとはいえ賃金が20円も増加するのだ。
これはshikakuとの対比で `20 / 1.5 = 13.33倍` となる。
十分に賃金が高くなり、少しのwork()で購入出来るのであれば積極的に購入するべきだ。

最終的にshikaku取得は `12.5 / 1.5 = 8.33work()` に収束する。
計算式(暫定)は `(8.33work() + 1purchase()) * 13.33倍 - 1purchase()` であり、
`1purchase() = 1work()` と仮定した場合、123.44work()となる。
つまり、123work()でprogrammingが買える状況下ならば買うべきだ。

### 上記を元に調査

この仮説＋ロジックは正しいのか軽く調査してみる。
とりあえずprogrammingは考慮に入れずに設定してみる。

```LiveScript
target = 1_0000_0000_yen
[1000 to 5000 by 1000] |> map ->
  shikaku: it
  cost: 10 * it + ((target / (2 + it * 1.5)) |> ceiling)
# [
#   {"shikaku":1000,"cost":76578},
#   {"shikaku":2000,"cost":53312},
#   {"shikaku":3000,"cost":52213},
#   {"shikaku":4000,"cost":56662},
#   {"shikaku":5000,"cost":63330}
# ]
```

どうやら2k〜3kの間が効率が良さそうだ。
この調子で `[1000 to 5000 by 1000]` の間を狭めていき、2570付近が最も効率が良いと仮定する。
実際はwork()とpurchase()のコストが完全に同じということは考えにくいので、ある程度上下させつつ様子を見るべきだろう。

```JavaScript
var purchases = (function(){
  var result = "";
  for (var i = 0; i < 2560; i=(i+1)|0) {
    result += "work();work();work();work();work();work();work();work();work();purchase('shikaku');";
  }
  return "(function(){ return function(){" + result + "}; })()";
})();
var works = (function(){
  var result = "";
  for (var i = 0; i < 2520; i=(i+1)|0) {
    result += "work();work();work();work();work();work();work();work();work();work();";
  }
  return "(function(){ return function(){" + result + "}; })()";
})();

var do_purchase = eval(purchases);
var do_work = eval(works);

start();
do_purchase();
do_work();
```

実際に測ったら2560前後が最速となるようだ。

```JavaScript
var purchases = (function(){
  var result = "";
  for (var i = 0; i < 640; i=(i+1)|0) {
    result += "work();work();work();work();work();work();work();work();work();purchase('shikaku');";
  }
  return "(function(){ return function(){" + result + "}; })()";
})();
var works = (function(){
  var result = "";
  for (var i = 0; i < 630; i=(i+1)|0) {
    result += "work();work();work();work();work();work();work();work();work();work();";
  }
  return "(function(){ return function(){" + result + "}; })()";
})();

var do_purchase = eval(purchases);
var do_work = eval(works);
var hatarake = [do_purchase, do_purchase, do_purchase, do_purchase, do_work, do_work, do_work, do_work];

start();
hatarake.forEach(function(it){it();});
```

### その後
この場合の賃金は2500*1.5=4k程度となるので、
プログラミングの初回は無理なく購入出来る。

上記コードにwork()*5、purchase(4)を付与して確認した所、
著しく実行時間が増えてしまった。

これは憶測の域を出ないが欲を出してプログラミングを購入した場合、
最初の購入タイミングで実績が解除され、そのレンダリングで大幅にロスとなってしまうようだ。
マシンスペック等にもよるかもしれないが…

