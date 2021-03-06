---
title: 【日記】Gauche の勉強の進捗＋その他
date: 2016-07-25T00:20:12+09:00
tags:
    - Diary
id: diary
---

今日はGaucheの勉強と買い物、掃除等を行った。
新言語ということでどんどん日記を書き出していこうと思う。

<!-- more -->

## 元々のプログラミング言語に対する考え

元々プログラミング言語は適材適所である。
Linuxのコア機能として作られている最速の高級言語であるC言語があるが、
低機能であったり、文法が未成熟でバグが出やすい仕様であったり、
メモリやポインタの事を常に意識する必要がある。

昔はPCのスペックが悪く、より低階層に触れる実行効率の良い言語である必要があったが、昨今ではその問題が解消してきた。
Ruby や Python 、Scala 等の良い言語の登場により、格段に読みやすくなり、バグも出づらくなったと感じる。
もちろん私が職場で使っている LiveScript もその一つで、PHPした知らなかった自分の世界が大きく広がったきっかけとなった。

これからの世界は多少実行効率を犠牲にしても、
可読性やバグの少なさ、シンプルさ…等の要素の強い言語に置き換わっていくだろう。

先日まではこのように考えていた。

## Scheme / Gauche と出会ったきっかけ

知人が LISP にハマった。
彼が言うには、世の中のハッカー達は LISP で仕事をしているらしい。
LISP のマクロを組み、Java のコード等に置き換えてそれを納品しているというのだ。

それほどまでに簡潔で美しい関数型言語があるとのこと。
詳しく話を聞くと、その中でも Scheme の Gauche を使い出したと聞く。
軽く調べた感じでは、Common LISP や Clojure と並ぶ三大LISPの一つ（自分の勝手な感想です）らしい。

Common LISP は実装を拡張してきた経緯からごちゃごちゃしている箇所もあるらしく、
新しく始めるなら Scheme や Clojure がお勧めらしい。
自分も友人も Java や JVM にはさっぱり興味がないので、必然的に Scheme を選択という流れだ。

とりあえず REPL を入れると勉強が捗ると教わり、
Gauche というREPLをインストールした。
Macbookに入れるのは一瞬だった。

```bash
brew install gauche
echo '(print (list->string (reverse (string->list "アルゴリズム"))))' | gosh
ムズリゴルア
```

ああ、これは簡潔で美しい言語だ。
一発で LISP や Scheme の持つ魅力に取り憑かれてしまった。

LiveScript を業務で使う上で、こうあるべきというソースコードは大抵こんな形で表現されている。
ダメな箇所は大抵が継承しまくったオブジェクト指向で書かれており、肝心のコードが何処かプロジェクト内を全文検索するハメになる。

早く書くには簡潔な言語が最も適している。
LISPの勉強は続けて行きたい。

## さて、勉強開始

注文した書籍は土日で到着するかもと思ったが、来週になる見込み。
注文したのは[プログラミングGauche](https://www.amazon.co.jp/gp/product/4873113482/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)という8年前の書籍

やりたいことは LiveScript で元々書いていたコマンドラインツールを Gauche に移植したいのだが、
SQLite や HTTPリクエストを投げたいので外部ライブラリに頼る事になりそう。
しかし、外部ライブラリの読み書きやパッケージマネージャーの事もよく知らないので、まずはリファレンスをななめ読みすることにした。

[Gauche ユーザーリファレンス 3.3 Schemeスクリプトを書く](http://practical-scheme.net/gauche/man/gauche-refj_15.html#Scheme_00e3_0082_00b9_00e3_0082_00af_00e3_0083_00aa_00e3_0083_0097_00e3_0083_0088_00e3_0082_0092_00e6_009b_00b8_00e3_0081_008f)
ここまで読んだ。（もちろん分からない箇所はとばしとばしだが…）

タイピング練習も兼ねてコードを写経したが、こんな簡潔にコマンドラインツールが出来るものなのかとびっくりした。
（LiveScript でもそれなりの効率を出せるが、LISP の効率の高さには一歩及ばずといったところだろう…）

## 他

ニトリに行った。
電動のベッド（最安値4万弱）とベッド用テーブルを見つけた。
本日はベッド用テーブルのみを購入した。
記事を書いているが少々腰が辛い印象…やはり電動ベッドが必要か。

