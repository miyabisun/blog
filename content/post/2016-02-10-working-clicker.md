---
title: 【働クリッカー】0.25秒達成
date: 2016-02-10 17:06:09
tags:
  - 働クリッカー
  - JavaScript
  - アルゴリズム
id: working-clicker
---

## 試行錯誤の結果

何回かやって得られた感想を箇条書き

- start関数を叩くまでタイムがスタートしない
  - 初期処理のコストは実質0
  - 関数や変数等の準備を十分行ってから望む事が出来る
- cash、items、pricesは単なる変数に思えるが上書きできない
  - ES6で実装された[getter](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/get)を使っているものと思われる
  - 本気でハックすれば突破出来る？…多分不可能なのだろう
- 用意されているloop関数は超絶遅い
  - 2番目の引数がms単位でsetIntervalと同じ作りだと思われる
  - 製作者が発表している0.8sですら、1msに1度しか実行できないloop関数では800回程度しか実行出来ない
  - 800回のwork()で1億とか到底無理で、関数内でfor文を回して1000回程度のwork()を叩きまくる糞関数を定義することになる
  - つか、それがまかりとおるなら最初からfor文で回せばよくね？
- 1秒以内に何万とwork()関数を実行する為、shikakuとprogramming以外は取る必要無し
- if文は遅い
  - 条件判定式自体のコストはそれほどではないのだが、何万work()を実行する事になるのでその度に調査していては無駄。
- for、whileと様々なループ文を試してみたが、再帰関数が最速なようだ
  - あくまでElectronで使われているV8のエンジン固有の話だと思われる

<!-- more -->

## これが多分最速だと思います

うちの環境(MacbookPro Retina13インチ)で0.24秒をマークした。

```JavaScript
works_num = function(num) {
  return function(){
    for (var i = 0; i < num; i++) {
      work();
    }
  }
};
shikaku = function shikakutore(it, works, loop){
  return function(){
    for (var i = 0; i < loop; i++) {
      works();
      purchase(it);
    }
  };
};
shikaku_tore = shikaku(0, works_num(20), 25);
program_tore = shikaku(4, works_num(500), 1);

hatarake = (function(){
  function hatarake(){
    shikaku_tore();
    program_tore();
    cash < 100000000 ? hatarake() : null;
  }
  return hatarake;
})();

start();
hatarake();
```

## 宿題

空き時間を利用してこつこつチューニングしたが、
このやり方ではこれ以上の速度増加が不可能と判断。

ん、閃いた！これやれば絶対最速に1億稼ぐロジックになるわ。
働クリッカーの動作を解析すればいけるな。
詳細は別記事で公開しようと思う。

