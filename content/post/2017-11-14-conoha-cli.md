---
title: ConoHa CLI 開発日誌 vol.1
date: 2017-11-14T21:53:40+09:00
tags:
    - Golang
    - ConoHa
id: conoha-cli
---

Go言語でConoHa-CLIを作成しているので、
少しずつ進捗報告をしていきたい。

## 現状

- Glideを導入
- cobra / viperを導入
- TOMLを導入
- 構造体の外出しでハマる

<!-- more -->

## Glideを導入

[Glide](https://github.com/Masterminds/glide)はパッケージ管理ソフト。
先日参加した[GoビギナーズLT大会！ 「最近、Go言語始めました」の会 #4](https://go-beginners.connpass.com/event/64866/)というイベントで、
Go言語のパッケージ管理はGlideを使うのが筋が良いと聞いて導入を決意。
参考サイト: [Goビギナーズ - よくある3つの質問 - mom0tomo](http://go-talks.appspot.com/github.com/mom0tomo/go-present/2017/10/go-beginners/present.slide#1)

`go get`はローカルマシンに落としてしまうのでパッケージ管理というレベルじゃねーぞ状態だったが、
Glodeではプロジェクトルートに設定ファイル、vendorディレクトリに依存ライブラリが次々と担ぎ込まれ、
npmとほぼ同じ使用感で使う事が出来た。

ただ、LT大会を聞いている限り、Glideを使って入れたらハマったので結局`go get`でやりました。
使い方教えてという悲痛な声だらけだったので、酷いハマリポイントがあるんだろうなぁ（白目

## cobra / viperを導入

[cobra](https://github.com/spf13/cobra#working-with-flags)はCLI用のライブラリ。
Node.jsに於ける[Commander.js](https://github.com/tj/commander.js)みたいなもん。
Commander.jsはサブコマンドの実装で大ハマリしたが、cobraはわりと素直な動作だしドキュメントが充実してて書きやすい。

お供は[viper](https://github.com/spf13/viper)、蛇繋がりか。
こちらはJSON、YAML、TOML等の設定ファイルをよしなに展開してくれる。
また、cobraとの共存も見越しており、オプション設定にフックしてよしなに上書きすることも可能。

なにそれ便利過ぎィ！
ただし、環境変数や複数の設定ファイル読み込み、オプションのオーバーライド等の兼ね合いから、
ファイルへの書き込みはサポートしていない模様。
(なんかファイルに書き出したいという要望がプルリクに上がってて、マージされてるっぽいんだけど、masterにはないもよう)

ファイルへ吐き出す場合は、別途何かしらのライブラリを用意してきて勝手に出力するしかない。
ただまぁ、viperから最新の値を抜き出して上書き出来るから全くのゼロスタートというわけではないかな。

## TOMLを導入

[TOML](https://godoc.org/github.com/BurntSushi/toml)とはGitHubの人が提唱している設定ファイル。
IniとJSONの良いとこ取りといった感じ、インデント無しで配列やオブジェクトを宣言出来るため、ネスト地獄が起こらないのが主張。
最初はYAMLで書いてたのだが、どうもTOMLの方が便利そうなのでそちらを利用することに。

## 構造体の外出しでハマる

コンフィグファイルまわりは外出ししようとプロジェクトルートにはconfというディレクトリを作成して外出し。
どうしたものか、`conf.auth = xxx`がundefinedですとコンパイル通さなくなった。
しかしここはさすがのQiitaクオリティ、同じネタでハマった人を発見して無事解決。

参考サイト: [Goの構造体をpackageに外出ししてハマッたこと](https://qiita.com/zurazurataicho/items/4a95e0daf0d960cfc2f7)
