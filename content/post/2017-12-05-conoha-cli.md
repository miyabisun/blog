---
title: ConoHa CLI 開発日誌 vol.4
date: 2017-12-06T22:41:00+09:00
tags:
    - Golang
    - ConoHa
id: conoha-cli
---

ConoHa-CLIの進捗報告。
チラ裏成分多め。

車駆動開発だとGoの依存ライブラリを落とすだけで世界が止まる。
早くConoHa環境でオラオラ作業したいわぁ。

## 現状

- 切り出したログイン処理のリファクタリング
- コンパイルエラーとの戦い
- Windows用のホームディレクトリの調整
- Glideはオワコン
- ゴールをどうするか悩む

<!-- more -->

## 切り出したログイン処理のリファクタリング

切り出したログイン処理が汚いが諦めて布団に潜った所、
翌日の朝に閃いてリファクタリング、まだまだ拙いが良い感じの粒度で切り出せたように思える。

## コンパイルエラーとの戦い

`single-value context`というエラーが出て中々解消出来ない。
とりあえず色々ググっていたら、戻り値が2個のものを1個だけ受け取っていた事が原因らしい。
不要の場合でも`_`で受け取る必要があるのね。

全部のコンパイルが通ってログインが通った所で一旦コミットしてGitHubへプッシュ。

- [Go言語の気に入ったところ/気に入らなかったところ](http://hakobe932.hatenablog.com/entry/2013/12/21/143305)

## Windows用のホームディレクトリの調整

viperは`$HOME`というパワーワードをよしなに解釈してくれるので問題なさそうだが、
コンフィグファイルを読み書きしている箇所はosパッケージを利用しているので、
Windows環境では恐らくコンフィグファイルを読み書き出来ない不具合が登場しそう。

`go-homedir`というパッケージを用意して読み書きに対応する
まず[GitHubのプロジェクトルート](https://github.com/mitchellh/go-homedir)へジャンプ！
うわ、ReadmeにUsageないじゃんピンチ!
あ、でも英語読めば十分理解出来る。

一応関数の使い方を確認しておきたいので`golang go-homedir`でぐぐったらGoDocなるサイトが引っかかって理解出来た。
念のためプロジェクトのhomedir.goも確認、たったの100行じゃん楽勝。
確かにDirは引数無しのstring, errを返すし、Expandはstringを受け取りstring, errを返す。

```Bash
$ glide get github.com/mitchellh/go-homedir
```

- [Golang で filepath.Abs に ~(ホームディレクトリ) を指定した時にカレントワーキングディレクトリが先頭についてしまう](http://tkuchiki.hatenablog.com/entry/2015/07/31/184555)
- [package homedir - GoDoc](https://godoc.org/github.com/mitchellh/go-homedir)

## Glideはオワコン

Glideで入れたgo-homedirが使えないので使い方の確認を兼ねて探していたら、
[Glide から dep に移行せよ - Qiita](https://qiita.com/spiegel-im-spiegel/items/e931ad1a7565d02d179e) の記事でdepに移行せよとか煽られてる

この間教えて貰ったばっかりなのに！
いやまだわからん、公式で確認しておこう。
GlideのGitHubプロジェクトルートへ行ってみると……

> ### Golang Dep

> The Go community now has the dep project to manage dependencies. Please consider trying to migrate from Glide to dep. If there is an issue preventing you from migrating please file an issue with dep so the problem can be corrected. Glide will continue to be supported for some time but is considered to be in a state of support rather than active feature development.

意訳: Golang公式がdep出したのでこれで良くね？。暫くGlideも更新続けるけど出来るだけ早く移行してね。

やっぱりダメだぁ…おしまいだぁ…
どうやら簡単に移行できるようなので家の無線LANを使って移行。
電車のテザリングじゃ限界が見えてきたので家で導入。

やった事はGlideが作った`glide.yaml`, `glide.lock`, `vendor/`を全て削除して、`dep init`を打ち込むだけ。
Homebrewで入れようとしたらXCodeの最新じゃなきゃダメとか言われたので`go get`で導入した。

- [Glide から Dep への移行を検討する - text.Baldanders.info](http://text.baldanders.info/golang/consider-switching-from-glide-to-dep/)
- [【golang】パッケージ依存解決ツールdepが使えるようになったので使ってみた！ - エンジニアはこわくない](http://tsujitaku50.hatenablog.com/entry/2017/02/24/190354)
- [Goオフィシャルチーム作成の依存関係管理ツール dep を試してみた](https://dev.classmethod.jp/go/dep/)
- [依存関係管理ツールdep(golang) - Qiita](https://qiita.com/Azizjan/items/66564b5dc7597717932b)
- [Masterminds/glide - GitHub](https://github.com/Masterminds/glide)
- [golang/dep - GitHub](https://github.com/golang/dep)

## ゴールをどうするか悩む

ConoHa CLIのゴールやりたいことはある程度明確にできていると思う。
要するに、VagrantのConoHaプラグインがうんともすんとも言わないから仕方なく自分で実装するんだ。

別に自分はVimで開発出来るからRsyncが使えなくとも困らない。
ファイルの共有がしたければFishで補完の効くscpコマンドでやり取りすれば良いのだ。

さて、元々考えていた最終ゴールは`conoha ssh`でPecoが立ち上がり、
SSHコマンドを内部的に代行して接続しにいくことだ。
でもGoのCLIからはどうやって逃げれば良いんだろう…よくわからん。
Vagrantでは現に`vagrant ssh`とタイプするだけで実際にSSH接続出来るわけだよなぁ、Rubyで…Rubyで出来るならGoでも出来なくないと思うのだけどさっぱりわからない。

`conoha ssh | bash -c`等と入力しておいて、
予め標準出力を実行しますよと明示しておけば一応満たすことはできそうだ。
でもそんなにタイプしたくないなぁ…

一旦は`conoha ssh`系コマンドで下記のように実装していくかな。
`~/.ssh/config`にさえぶち込んでしまえばpecoで起動したり、scpの補完が効くからね。

- conoha ssh init: `~/.ssh/conf.d/`ディレクトリを作成。
- conoha ssh write: `~/.ssh/conf.d/conoha.conf`ファイルに登録済みのインスタンスへの接続情報を全て格納
- conoha ssh create: `~/.ssh/conf.d/*.conf`ファイルを結合して新しい`~/.ssh/config`ファイルを生成する
- conoha ssh update: write -> createの順に実行

