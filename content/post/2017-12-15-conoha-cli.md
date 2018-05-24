---
title: ConoHa CLI 開発日誌 vol.7
date: 2017-12-19T09:32:00+09:00
tags:
    - Golang
    - ConoHa
id: conoha-cli
---

ConoHa-CLIの進捗報告。
20連勤と毎日午前様のせいで全く進まないままアドベントカレンダー当日になってしまった…

結論から言うと、コピペコードでなんとか納めて公開したわけなんだが…

## 現状

- infoサブコマンドの実装
- imports を動的に生成
- コピペコードで実装（苦

<!-- more -->

## infoサブコマンドの実装

では早速作っていこう、
エンドポイントはこんな感じになるのかな。

```Go
var endpoints = map[string]string]{
    "images": "https://image-service.tyo1.conoha.io/v2/images",
    "flavors": "https://compute.tyo1.conoha.io/v2/1864e71d2deb46f6b47526b69c65a45d/flavors",
}
```

…ってのっけからテナントID求めるのかよ。
ん〜…%sで引っ掛けて`fmt.Sprintf`使うか

## imports を動的に生成

Vim使いなのでシンタックスハイライト用にvim-goを入れていたが、
どうやらimportsを自動的に表現してくれるコマンドがあるらしいとのこと。

[hnakamur/vim-go-tutorial-ja - GitHub](https://github.com/hnakamur/vim-go-tutorial-ja)

> インポートパスを手動で操作するのはとても 2010 年っぽい (訳注: 時代遅れ) です。この問題を扱うもっとよいツールを提供しています。もしまだ聞いたことがなければ それは `goimports` と呼ばれています。

> 私たちは `:GoImports` コマンド (最後の s に 注意) も用意しています。これで明示的に `goimports` を実行することができます。

これだね。

```
let g:go_fmt_command = "goimports"
```

これ入れておけば自動的にやってくれるみたいだから入れておこう。
不要になったら削除までしてくれるらしいから完璧だね。

さて、早速実験。
`fmt.printfn("hogehoge")`で保存っと、よしよしちゃんと`import "fmt"`になった。
じゃあ`conf.Read()`を使ったらどうなるのっと…`"github.com/miyabisun/conoha-cli/conf"`
嘘だろ……！？ これもバッチリ解決するとは恐ろしい……

もう私は長いGolang人生でimport文を自分で修正することは多分無い。
とはいえ、viperだけで"github.com/spf13/viper"を提示することは難しいと思うので、
この辺の依存関係をどう解決するかは今後も確認する必要がありそうだね。

## コピペコードで実装（苦

結局時間切れになってしまったので、
ConoHa CLIの実装をコピペで一気に行った。

結論から言えば新しい言語にフレたとき、
最初からこうしておけばよかったかも知れない。
なぜなら共通部品にしたい箇所が山ほどあり、改善案が湯水のように溢れ出してきたから。

ただし、これをもって完成とは決して言ってはならないし、
究極の目標はテスト駆動開発を身につける事であるので、初見のものも上手くテスト駆動で開発出来るようにしていきたい。

## 今後やりたいこと

`conoha ssh`は絶対に導入したい。
まずは`.config/conoha.toml`内にssh-keyとローカルの秘密鍵の対応表を盛り込むコマンドを実装しなければ…

