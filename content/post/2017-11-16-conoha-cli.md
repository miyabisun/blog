---
title: ConoHa CLI 開発日誌 vol.2
date: 2017-11-16 22:56:59
tags:
    - Golang
    - ConoHa
id: conoha-cli
---

今日のConoHa-CLIの進捗報告。
ConoHa-CLIの報告だと、上司はこのはちゃんになるのだろうか？

## 現状

- 標準入力からの値取得
- ConoHaのAPIを叩いてログイン(トークンID取得)
- トークン情報を引き出す
- TOMLファイルを更新

<!-- more -->

## 標準入力からの値取得

現在loginコマンドを実装中。
オプションでID、Password、テナントIDを入力させる作りにはしたが、
入力項目が足りない場合は標準入力から煽って入力させる作りを目指した。

参考サイト:

- [flag パッケージ - golang.jp](http://golang.jp/pkg/fmt)
- [Go 言語で標準入力から読み込む競技プログラミングのアレ --- 改訂第二版](https://qiita.com/tnoda_/items/b503a72eac82862d30c6)

早速参考サイトを元に実装を行う。
優先順位の要件は下記。

1. コマンドライン引数`-t`からの入力値
2. `$HOME/.conoha`の設定項目
3. 標準入力と標準出力で入力させる

まずはcmd/root.goでcobraとviperの設定を行う

```Go
func init() {
    cobra.OnInitialize(initConfig)
    RootCmd.AddCommand(versionCmd)
}

func initConfig() {
    viper.AddConfigPath("$HOME")
    viper.SetConfigName(".conoha")
    viper.SetConfigType("toml")
    viper.ReadInConfig()
}
```

次にcmd/login.goで各種値を取得するように設定を行う。

```Go
func init() {
    rootcmd.addcommand(logincmd)
    logincmd.flags().stringp("tenantid", "t", "", "port to run application server on")
    viper.bindpflag("auth.tenantid", logincmd.flags().lookup("tenantid"))
}

var loginCmd = &cobra.Command{
    Use:   "login",
    Short: "Calculator of addition.",
    Long:  "Calculator to perform the addition.",
    Run: func(cmd *cobra.Command, args []string) {
        tenantId := viper.GetString("auth.tenantId")
        if tenantId == "" {
            fmt.Print("tenantId: ")
            fmt.Scan(&tenantId)
        }
        fmt.Print(tenantId)
        # tenantId: test1
        # test1
    }
}
```

## ConoHaのAPIを叩いてログイン(トークンID取得)

URLにアクセスする所もよく分からんから困るなあ、
とりあえずぐぐってみるとnet/httpパッケージがあるらしい。

調査中にQiitaの記事のコメント欄で凄い事を聞いてしまった。

> shibukawa:
> 簡単なスニペットなら、僕が作ったこのサイトで作れますよ。Curlコマンドからgoのコードが生成できます。ぜひぜひ。
> [https://shibukawa.github.io/curl_as_dsl/](https://shibukawa.github.io/curl_as_dsl/)

貴方が神か。
早速[トークン発行ページ](https://www.conoha.jp/docs/identity-post_tokens.html)に記載してあるcurlコマンドのサンプルをベタり。
そのままでは通らんかったのでPOSTのパラメータを削るとGoのコードが吐き出されたので丸コピー。

削ったPOSTのパラメータは`fmt.Sprintf`を使って作り直した。
あっさりログイン成功し、こんなに簡単に出来て良いのだろうか。
実力はついてない気がしないでもないが…

ステータスコードが200以外ならエラー文言を画面表示したい。
公式のnet/httpパッケージを見ると`resp.StatusCode`がそれらしい。

- [Go net/httpパッケージの概要とHTTPクライアント実装例 - Qiita](https://qiita.com/jpshadowapps/items/463b2623209479adcd88)
- [Package http](https://golang.org/pkg/net/http/)
- [cURL as DSL](https://shibukawa.github.io/curl_as_dsl/)
- [【Go】print系関数の違い](https://qiita.com/taji-taji/items/77845ef744da7c88a6fe)

## トークン情報を引き出す

body部をStringに変換して表示するとJSONであった。
Qiitaの記事を参考にjson.Unmarshalで簡単に…取り出せない！？
俺は`access.token.id`を取り出したいだけなんだ！！

結局構造体を3個余分に作ってネストしていくように実装した。
この辺は要課題だね。

- [Go言語でJSONを扱う - Qiita](https://qiita.com/nayuneko/items/2ec20ba69804e8bf7ca3)
- [Goの構造体に付加するタグを複数にしたい場合 - Qiita](https://qiita.com/aibou/items/33142a988e8bd4a4d164)
- [Go言語で時刻を - そこはかとなく書くよん。](http://tdoc.info/blog/2013/04/10/go_time.html)
- [JSONのパース/生成 - はじめてのGo言語](http://cuto.unirita.co.jp/gostudy/post/standard-library-json/)

## TOMLファイルを更新

ここまでくれば余裕だろ。なに？ファイルに書き込めないだと？
どうやら`$HOME`をよしなに解決してくれるのはViperの便利機能であって、自前でファイルに書き込む時はenvやosパッケージを参考に引っ張ってくる必要があるらしい。

- [BurntSushi/toml - GitHub](https://github.com/BurntSushi/toml)
- [ファイル入出力 - はじめてのGo言語](http://cuto.unirita.co.jp/gostudy/post/standard-library-file-io/)
- [Package os](https://golang.org/pkg/os/#FileMode)
- [Obtain user's home directory](https://stackoverflow.com/questions/7922270/obtain-users-home-directory)
- [ホームディレクトリを取得するのにos/userを使うとエラーになる場合がある - Qiita](https://qiita.com/hironobu_s/items/da2f97c2154075d3fbbe)

## 実はviperも読み込めてませんでした

最初は`$HOME/.conoha`にしようと思ってたのだが、
viperのconfigファイルは開幕ドットの隠しファイルはNGらしい。
設定次第で行けるのか？よくわからん…苦戦の末`$HOME/.config/conoha.toml`で妥協。

しかし、美雲このはからコノハ＝トムルに改名する緊急事態である。
<del>面白そうだからいいか</del>

- [Multiple config paths doesn't seem to be working #104](https://github.com/spf13/viper/issues/104)
- [Golang の文字列連結はどちらが速い？](https://qiita.com/spiegel-im-spiegel/items/16ab7dabbd0749281227)
- [Go言語入門(ファイル・ディレクトリ操作)](https://qiita.com/knt45/items/557ee65c46a685ea4f59)

## Next

ログインコマンドは概ね実装が完了したので、次回はモジュールの整理。
今回のJSON解析でトークンの有効期限を文字列で取得することに成功した。
次は一度トークンを確認してから実行するようにしたい。

- [Go言語で時刻を - そこはかとなく書くよん。](http://tdoc.info/blog/2013/04/10/go_time.html)
