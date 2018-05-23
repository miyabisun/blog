---
title: 【MySQL】再帰問い合わせ
date: 2012-10-18 14:26:26
tags: MySQL
id: recursive-mysql
---

## なぜ再帰問い合わせが必要になったか

うちのブログがidとparent_idで木構造を表現しているが、
ぶら下がってる全ての記事一覧を取得したい。

それにはSQLで定義されているWith句を利用すれば良いが、
MySQLではまだ未実装らしい(2011/02時点)

しかし、再帰問い合わせなんて単語を知らなかったので、
調査ついでにまとめてみた。

<!-- more -->

## WITH句に関しての説明

新しい業界標準「SQL99」詳細解説
<http://www.atmarkit.co.jp/fnetwork/tokusyuu/01sql99/sql99_1b.html>

## ぐぐった結果

### PHPとMySQLでツリー構造を扱う

<http://aratanuki.blog100.fc2.com/blog-entry-82.html>
これはPHPでクエリーを都度投げる方法。
この記事では`mysqli::prepare()`を使っている。
これを参考に実装するならば、まず親記事の一覧を作成し、
「WHERE parent_id IN (親記事の一覧)」として記事の一覧を取得してくる感じか…

### 子要素を再帰的に取ってくるストアドプロシージャ

<http://blog.udzura.jp/2010/08/21/recursive-stored-procedure/>
ストアドプロシージャ？
一時的にテーブルを作成し、結果をどんどん投げ込んでから再度検索しなおす感じか？

## 結論

MySQLで再帰問い合わせは難しい
ブログに使うのは諦めた

