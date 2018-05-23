---
title: 【ZendFramework】インサートしたIDを取得
date: 2013-06-10T11:03:57+09:00
tags:
  - PHP
  - MySQL
id: mysql-insert
---

## 最後にインサートしたレコードのIDを取得する方法
オートインクリメントで振られたIDを知りたい時の話

## 普通の方法
大抵のRMDBはオートインクリメントで振られたIDを覚えているので、
再度クエリを発行すればID値を取り出すことができる。

- 複数のユーザが同時にアクセスして登録したら？
  - 軽く試してみたが問題ない。
    - コネクションを張る度に別プロセスと認識される
    - 同一コネクション内でインサートしたオートインクリメント値が還ると思われる

### MySQL
```SQL
SELECT last_insert_id()
```

### SQLite
```SQL
SELECT last_insert_rowid()
```

<!-- more -->

## PHPによるアダプターを利用する場合

### PDO
`PDO::lastInsertId`を使う。
上記のように、オートインクリメントによって振られたIDを取り出すSQL文はDB毎で異なる為、
PHPのソースコードとDB操作を分けるには不向きで、その差異を吸収する為に存在するメソッドと思われる。
要するにPDOを使う場合は何も考えずに`PDO::lastInsertId`でおｋ

```PHP
// DBへ接続し、INSERT文を発行
$db = new PDO($dsn);
$sql = 'INSERT INTO .....';
$db->exec($sql);

// オートインクリメント値を取り出す
$id = $db->lastInsertId();
```

### Zend Framework
アダプタークラスに`lastInsertId`というメソッドが用意されている。
アダプターにPDOを使用した場合、`lastInsertId`を呼んでるらしい。
```PHP
$table = new Zend_Db_Table();
// Insert文発行
$adapter = $table->getAdapter();
$id = $adapter->lastInsertId();

$id = $table->getAdapter()->lastInsertId(); // 一発でやってもOK
```

`Zend_Db_Table`オブジェクトにアクセス出来れば`lastInsertId()`メソッド叩くだけで出てくる。
自社開発のコレクターとかモデルとか使ってる場合でも、
以下のように`Zend_Db_Table`オブジェクトを返すメソッドを作成すれば良い。

```PHP
$id = $db_model->getTable()->getAdapter()->lastInsertId();
```

