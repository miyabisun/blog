---
title: CoreOSのリブートを止める
date: 2016-01-21T11:16:58+09:00
tags:
  - CoreOS
  - systemd
id: coreos-memo
---

## 対応策

CoreOS - Reboot Strategies on Updates
<https://coreos.com/os/docs/latest/update-strategies.html>

### Disable Automatic Updates Daemon

ここの項目を読み進めると下記のコマンドを入力すると書いてある。

```Bash
$ sudo systemctl stop update-engine
```

<!-- more -->

## 絶対バッドノウハウだろ！？

…はい。
でも、もう稼働中の本番環境なんです…
ダウンするわけには行かないんです…

本来はダウンしても大丈夫なようにシステムを構築するべきで、
絶対にあるべき姿じゃない、この方法は応急処置にとどめておいてください…

※ちなみに現在根本的な対応中です。

## 何故勝手に落ちるのか？

CoreOSの設計思想として、
セキュリティホールを放置したまま延々と起動し続けて全世界の皆様に迷惑をかけるくらいなら、
さっさとアップデートして再起動しろやということなんだろう（憶測）

### どのプロセスが原因なのか

はてぶ：tmatsuuのコメント
<http://b.hatena.ne.jp/entry.touch/194619714/comment/tmatsuu>

> CoreOS再起動とfleetがまだ使えない件は同じくハマった。
> 自動再起動の止め方、昔と違ってた。
> 詳しくは公式ドキュメントのUpdate Strategiesを参照。

神様ありがとう！
どれどれ…

Reboot Strategies on Updates
<https://coreos.com/os/docs/latest/update-strategies.html>

「Disable Automatic Updates Daemon」の項目に
`sudo systemctl stop update-engine`としてくれと書いてある。

### どんな設定か確認してみる

この辺はほぼ初期状態のままだから
公開してもリスクはないと考えて公開。

CoreOSのサービスデーモンはsystemdで管理しているので、
まずはUnitファイルを確認。

```Bash
$ systemctl cat update-engine.service
# /usr/lib64/systemd/system/update-engine.service
[Unit]
Description=Update Engine
ConditionVirtualization=!container
ConditionPathExists=!/usr/.noupdate

[Service]
Type=dbus
BusName=com.coreos.update1
ExecStart=/usr/sbin/update_engine -foreground -logtostderr
BlockIOWeight=100
Restart=always
RestartSec=30

[Install]
WantedBy=default.target
```

引き続きロード済のUnitファイルがどう使われているか確認

```Bash
$ systemctl status update-engine
● update-engine.service - Update Engine
   Loaded: loaded (/usr/lib64/systemd/system/update-engine.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2015-12-12 00:46:27 UTC; 1 months 9 days ago Main PID: 703 (update_engine)
   Memory: 11.2M
      CPU: 2min 55.787s
   CGroup: /system.slice/update-engine.service
           └─703 /usr/sbin/update_engine -foreground -logtostderr
```

CoreOSの公式ドキュメントにも書いてある
`sudo systemctl stop update-engine`でとりあえず再起動は防げる事がわかる。

## 再発防止策(試行錯誤中)

### NG: Unitセクションの動作除外条件を利用

`[Unit]`セクションに
`ConditionPathExists=!/usr/.noupdate`なんて項目があってあからさまに怪しい。

kanda.motohiro: man systemd.unit の訳
<https://sites.google.com/site/kandamotohiro/systemd/man-systemd-unit-no-yi>

> ConditionPathExists= は、ユニットが開始する前に、あるファイルが存在するかチェックする。
> 指定された絶対パス名が存在しない場合、条件は偽となる。
> ConditionPathExists= に与えられた絶対パス名が感嘆符 ("!") でプレフィックスされている場合、条件は逆に評価され、ユニットはそのパスが存在しない場合に限り開始する。

ドンピシャである。
ファイルを作って余裕よゆ…

```Bash
$ sudo touch /usr/.noupdate
-bash: .noupdate: Read-only file system
```

`/usr`の直下だしsudo使ってもダメか…

### NG: Unitファイル書き換え

どう見ても真似するな危険級なのを承知で一応試してみる。

`systemctl status`を打ち込んだ時に、
読み込み元のUnitファイルのパスが表示されてた…
ということで実体を書き換えようとしてみた。

```Bash
$ cat /usr/lib64/systemd/system/update-engine.service
[Unit]
Description=Update Engine
ConditionVirtualization=!container
ConditionPathExists=!/usr/.noupdate

[Service]
Type=dbus
BusName=com.coreos.update1
ExecStart=/usr/sbin/update_engine -foreground -logtostderr
BlockIOWeight=100
Restart=always
RestartSec=30

[Install]
WantedBy=default.target
```

なるほど、`/usr/lib64/systemd/system/update-engine.service`が実体で間違いないな。
とりあえずバックアップを取得して…弾かれた。
もちろん、この後`update-engine.service`ファイル自身を書き換えようとしたがやはり弾かれた。

```Bash
$ cd /usr/lib64/systemd/system
$ sudo cp update-engine.service update-engine.service.bak
cp: cannot create regular file 'update-engine.service.bak': Read-only file system
```

### NG: `systemctl disable`を使う

「Systemd」を理解する ーシステム管理編ー | ギークを目指して
<http://equj65.net/tech/systemd-manage/>

`systemctl is-enabled [Unit名]`で調べられるとのこと。

```Bash
$ systemctl is-enabled update-engine
disabled

$ sudo systemctl disable update-engine
$
```

なん・・・だと・・・？
最初からdisabledだった。
ちなみに一度立ち上げると標準出力の内容が出力され、どう設定したか表示してくれるようだ。

```Bash
$ systemctl is-enabled update-engine
disabled

$ sudo systemctl enable update-engine
Created symlink from /etc/systemd/system/default.target.wants/update-engine.service to /usr/lib64/systemd/system/update-engine.service.

$ systemctl is-enabled update-engine
enabled

$ sudo systemctl disable update-engine
Removed symlink /etc/systemd/system/default.target.wants/update-engine.service.

$ systemctl is-enabled update-engine
disabled
```

調査すると最初からDisabledの理由が分かった。

AWS+CoreOS+Dockerでコンテナの自動起動 1 | Qiita
<http://qiita.com/aki/items/979b25ff555eb7ab96fc>

```
systemctl enable/disableは
/etc/systemd/system/multi-user.target.wants/
ここにシンボリックリンクを張ったり消したりするだけのようだ
```

systemdの機能により実行されたわけではなく、
CoreOSが新しく立ち上げている機能のようだ。

