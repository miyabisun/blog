---
title: ConoHa CLI 開発日誌 vol.8
date: 2017-12-21T23:01:00+09:00
tags:
    - Golang
    - ConoHa
id: conoha-cli
---

ConoHa-CLIの進捗報告。
まだまだやりたいことには届いてないから引き続き考えていくよ！

## 現状

- Golangのパッケージに関して
- 複数環境のリリースに関して
- SSHに関して

<!-- more -->

## Golangのパッケージに関して

Node.jsのrequireは1ファイル内で`module.exports`に記載したものだけ返すが、
Golangのパッケージは1ディレクトリを1パッケージとして、全角英字で始まる関数やインスタンス等が全て返ってくる。
抽象化は1度しか行えないのでしっかりとオブジェクト指向的な設計が必要になる。

今回Go初挑戦ということで適当に作ったパッケージがあまり使い物にならないので見直す必要が出てきた。
現状の問題点としては、conf.Read()関数がconfigファイルを返す為、spec.tomlを操ろうと思ったらRead関数が使用済みの為に困ってしまう。

一旦はこの2つにリファクタリングしよう。

- config
- endpoints

書籍を購入して勉強しているのだが、ディレクトリによるパッケージはサブディレクトリを切ればネスト出来るのでNode.js的な管理方法も十分可能に思える。
この辺を視野にいれて調査していこう。

参考サイト:

- [パッケージについて - はじめてのGo言語](http://cuto.unirita.co.jp/gostudy/post/go-package/)
- [Go言語入門](https://www.amazon.co.jp/Go%E8%A8%80%E8%AA%9E%E5%85%A5%E9%96%80-%E9%A3%9B%E6%9D%BE-%E6%B8%85/dp/4865940413/ref=sr_1_12?ie=UTF8&qid=1513864570&sr=8-12&keywords=Go%E8%A8%80%E8%AA%9E)

## 複数環境のリリースに関して

GitHubではタグを打った後、ファイルをアップロードすることでバイナリファイルをGitHub上に設置出来るらしい。
64bitのWindows、Mac、Linuxの3つくらいは上げておきたいから勉強必須。
ゆくゆくはMakefileかなんかで一撃でリリース出来るようにしたい。

参考サイト:

- [Go のクロスコンパイル環境構築 - Qiita](https://qiita.com/Jxck_/items/02185f51162e92759ebe)
- [privateなGitHub Releaseページのリリース物をcurl+jqでダウンロードするワンライナー - Qiita](https://qiita.com/minamijoyo/items/0affb46414cb746438bc)
- [GitHubのリリース機能を使う - Qiita](https://qiita.com/todogzm/items/db9f5f2cedf976379f84)
- [Creating Releases - GitHub Help](https://help.github.com/articles/creating-releases/)
- [Release Your Software](https://github.com/blog/1547-release-your-software)

## SSHに関して

GolangにはSSHパッケージがデフォルトで用意されており、osの標準入力/出力につなげてやると簡単に実装出来るとのこと。
これによりWindows環境でも（多分）動作するSSHになりうる。

でもメインが電車によるSSH接続なのでMoshにも対応したいなあ。。。
というわけで、もう少し調査した後に再開という形になる。

参考サイト:

- [Windows からも ssh でリモートコマンド実行したい、それ golang で出来るよ](https://mattn.kaoriya.net/software/lang/go/20170111165324.htm)
- [artyom/mosh - GitHub](https://github.com/artyom/mosh)
- [StanfordSNR/guardian-agent - GitHub](https://github.com/StanfordSNR/guardian-agent)
