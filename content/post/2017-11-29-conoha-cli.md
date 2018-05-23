---
title: ConoHa CLI 開発日誌 vol.3
date: 2017-11-29T22:14:56+09:00
tags:
    - Golang
    - ConoHa
id: conoha-cli
---

ConoHa-CLIの進捗報告。
まだちゃんと全てを切り出せたわけではないが、

## 現状

- ログイン処理の外出し
- 時刻の比較
- ログインの再実行

<!-- more -->

## ログイン処理の外出し

ログインコマンドは出来上がった。
ConoHaのAPIへ接続し、TokenIdを持ち帰り`~/.config/conoha.toml`に吐き出せるようになった。
しかし、このTokenIdはわずか1日しか有効期限がない上、APIの殆どはTokenIdを求めてくる。

現状の作りでは日付を跨ぐ度にTokenIdの有効期限が切れていますエラーを食らい、
その度にログインコマンドを打ち込む作業が待っている。

ログイン処理を抽象化して取り出す必要がある。
早速プロジェクトルートに`util`というディレクトリを作成した。
(ベストプラクティスはまだ良くわかっていないが、公開して有識者の目に止まればPRでも貰えるだろう)

- [Go言語でエラーを書くときに気をつけること - taknb2nchのメモ](http://d.hatena.ne.jp/taknb2nch/20140307/1394169432)
- [Go言語でパッケージの修飾名が重複した場合の対応方法 - taknb2nchのメモ](http://d.hatena.ne.jp/taknb2nch/20140211/1392046399)

## 時刻の比較

まずは時刻。
トークン時刻が期限切れか否かを確かめる方法がよく分からない。
Time同士の比較はないのか？
一度Unixタイムに変換して数値同士で比較するように修正した。

- [Package time](https://golang.org/pkg/time/#Unix)
- [Go言語で時刻を - そこはかとなく書くよん。](http://tdoc.info/blog/2013/04/10/go_time.html)
- [逆引きGolang (日付と時刻)](https://ashitani.jp/golangtips/tips_time.html#time_Unix)
- [How to convert ISO 8601 time in golang?](https://stackoverflow.com/questions/35479041/how-to-convert-iso-8601-time-in-golang)

## ログインの再実行

トークンが古ければもう一度ログインを試みて取得しなおすように修正。
Golangだと処理的になってしまうなぁ……色々とださいがまぁまぁええわ。

動くの大事。
最低限体裁を整えたので一度完了として次へ

## Next...

Mac以外のHomedirectoryも取得出来るようにしたい

- [Golang で filepath.Abs に ~(ホームディレクトリ) を指定した時にカレントワーキングディレクトリが先頭についてしまう](http://tkuchiki.hatenablog.com/entry/2015/07/31/184555)
- [package homedir](https://godoc.org/github.com/mitchellh/go-homedir)
