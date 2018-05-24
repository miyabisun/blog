---
title: ConoHa CLI 開発日誌 vol.5
date: 2017-12-09T11:22:47+09:00
tags:
    - Golang
    - ConoHa
id: conoha-cli
---

ConoHa-CLIの進捗報告。
個々の所午前様ばかりで全然作業が進まない。

今回はしっかり時間を作ってどう作って行くかを考えていきたい。

## 現状

- ConoHaのゴールに関して
- カレントディレクトリのファイル仕様
- コマンド一覧（予定）

<!-- more -->

## ConoHaのゴールに関して

前回考えたConoHaのゴールに関して、`vagrant ssh`コマンドが再現出来れば良い事はわかっている。
諦める前に一回確認しておこう、というわけでGitHubに公開されているソースコードを閲覧してみた。
該当するスクリプトはこの辺にありそうだ。

- [vagrant/lib/vagrant/action/builtin/ssh_exec.rb](https://github.com/hashicorp/vagrant/blob/master/lib/vagrant/action/builtin/ssh_exec.rb)
- [vagrant/lib/vagrant/action/builtin/ssh_run.rb](https://github.com/hashicorp/vagrant/blob/master/lib/vagrant/action/builtin/ssh_run.rb)
- [vagrant/lib/vagrant/util/ssh.rb](https://github.com/hashicorp/vagrant/blob/master/lib/vagrant/util/ssh.rb)

まぁ、Rubyのコードをあまり読めないから大変なんだが、どうやら`vagrant/action/ssh_run.rb`を読む限り、具体的な処理は`util/ssh.rb`に移譲しているようだ。
うーむ、やはりサブプロセスで動作させて、インタラクティブな状態で紐付けているようだ。
しかしこれでSSHで繋いだ先でVimやtmuxを平然と使えるって凄いな。

なんとなくできそうな事はわかったが、Goだとどうやって実装すればいいんだ。

- [How can I open an interactive subprogram from within a go program?](https://www.reddit.com/r/golang/comments/2nd4pq/how_can_i_open_an_interactive_subprogram_from/)
- [josephspurrier/sshclient.go - GitHub Gist](https://gist.github.com/josephspurrier/e83bcdbf9e6865500004)

ん？なんかめっちゃ簡単な事書いてないか？

```Go
cmd := exec.Command("ssh", option)
cmd.Stdout = os.Stdout
cmd.Stdin = os.Stdin
cmd.Stderr = os.Stderr
cmd.Run()
```

まさかこれだけでいける？
オプションのあたりは作り込む必要がありそうだけど、一応覚えておこう。
既存sshと連携したければvagrantと同じく`conoha ssh-config`で表示されるようにしたい。

## カレントディレクトリのファイル仕様

- spec.toml: 仕様(名前, OS, ssh-key, インスタンスタイプ...などなど)
- instance.toml: ConoHaのインスタンスの情報

1ディレクトリ、1設定ファイル、1インスタンス。
これもvagrantにある程度合わせる形で考えた。

複数端末でログインした場合でも、
spec.tomlを元にConoHa APIに問い合わせて最新のインスタンスの状態を取得出来るから。
instance.tomlは.gitignoreに放り込んでおく事で、spec.tomlをGitHubで管理出来るという狙いもある。

そうそう、ConoHaには初回インスタンスを立ち上げた時、
URLからスクリプトファイルを実行する機能もあるから上手く活かしたいね。

## コマンド一覧（予定）

- conoha login: ConoHa APIへログインして再ログイン情報やトークンを~/.config/conoha.tomlへ保存
- conoha init: カレントディレクトリへspec.tomlのスケルトンを作成する
- conoha info images: 利用可能なイメージ一覧を表示
- conoha info ssh: ConoHa に登録済みのis_rsa.pubを表示
- conoha status: インスタンスの状態を確認
- conoha up: インスタンスの起動
- conoha destroy: インスタンスの削除
- conoha suspend: インスタンスの停止（VPSなので課金は継続）
- conoha ssh: インスタンスへのSSHアクセス
- conoha ssh-config: インスタンスへ接続するSSHアクセス情報を表示
- conoha ssh-set: ConoHaのis_rsa.pubとローカルid_rsaを紐付ける

## まとめ

要件多すぎぃ

というわけで一箇所に絞って開発していくか。
最初は`conoha up`、`conoha ssh-config`がゴールになるだろう。
過程でインスタンスを構成する情報が欲しいから`conoha info`系を実装することになるだろうけどん。
