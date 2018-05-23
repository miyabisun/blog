---
title: 【MySQL】インデックスとは
date: 2012-11-14 15:54:29
tags: MySQL
id: mysql-index
---

## 目的

MySQLのインデックスに関して理解する。
- インデックスとはなんぞや？
- 使い方の学習
- 実際に使ってみる＆既存システムを覗いてみる

<!-- more -->

## インデックスとはなんぞや？

検索が早くなる位しか知らないので、
何故通常の検索では遅いのか、何故インデックスを使用するのか。
その辺から勉強していきたい（今更）

MySQLパフォーマンスチューニングのためのインデックスの基礎知識
<http://d.hatena.ne.jp/kiyo560808/20101117/1289952549>

> レコードの数が増えるにつれ、
> 特定のレコードを見つけ出すのに必要な処理が増える。
> これをO(n)問題という。

- メリット
  - 検索する時にインデックスのみを見て式を評価
  - 高速化が期待出来る
-デメリット
  - 所持するデータを複数持つので容量が肥大化
  - 書き込み(SELECT以外)のパフォーマンスが低下

### 速度に関して

じゃあ実際の速度ってどうなの？
あんまり速くならないならデータを冗長化させる必要なんてないしね

ソーシャルゲーム開発者なら知っておきたい MySQL INDEX + EXPLAIN入門
<http://www.infiniteloop.co.jp/blog/2011/03/mysql-index-explain/>

MySQL INDEX+EXPLAIN入門 のスライド5枚目

> インデックスを使用すると、
> SELECTに掛かる時間がレコードに比例せず一定

(  Д ) ﾟ ﾟ
ただ、この最小単位が100万レコードってのが一つのキーになりそうだ。
1万程度までならインデックスを作成しない選択肢もあり得るのかもしれない。

でも、別にインデックス作ったからと言ってSelectが遅くなる訳じゃないので、
理由が無ければとりあえず作る位の考えでも良いのか？

## 使い方の学習

上記サイトに書いてあった方法
`ALTER TABLE foo ADD INDEX (id, name)`

効果的な使い方のキモは上記2サイトを参考
- インデックスの作成時
  - 選択性の高い項目をインデックスにする
  - 複合インデックスは先頭から順に部分インデックスとしても使用できる
  - インデックスサイズを増やさない・増えることによる影響

一例：MySQLでインデックスを使って高速化するならCovering Indexが使えそう
<http://blog.livedoor.jp/sasata299/archives/51336006.html>

```SQL
-- インデックスを作成していない場合 (2.99 sec)
SELECT SQL_NO_CACHE hoge, fuga FROM table_1 WHERE foo = 1;

-- インデックス index_1 の作成
ALTER TABLE table_1 ADD INDEX index_1(foo);

-- インデックス index_1 を利用した場合 (2.52 sec)
SELECT SQL_NO_CACHE hoge, fuga FROM table_1 WHERE foo = 1;

-- インデックス index_2 の作成
ALTER TABLE table_1 ADD INDEX index_2(foo, hoge, fuga);

-- インデックス index_2 を利用した場合 (0.01 sec)
SELECT SQL_NO_CACHE hoge, fuga FROM table_1 WHERE foo = 1;
```

(  Д ) ﾟ ﾟすげえ
とりあえずこの方式は覚えておこう

