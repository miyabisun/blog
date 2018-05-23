---
title: Macbook Pro を再セットアップ
date: 2017-03-04 14:15:11
tags:
  - Diary
id: macbook-setup
---

仕事で使っていたMacbook Proを使用しなくなったので、
クリーンインストールして持って帰っていたのだが、
仕事が忙しくてさっぱり触れてなかったので再インストールした。

<!-- more -->

## ソフトの導入

### テンプレ

- [iTerm2](https://www.iterm2.com/): 必須ターミナル
- Alfred2: クリップボード履歴用
- Flexiglass: ウィンドウのりサイズ

### Homebrew

まずは何も考えずにXcodeをインストールする。
[Homebrew]はC++のバイナリを持ってきてMac内でコンパイルする仕組みだからXcodeがどうしても必要。
Xcodeの認証を済ませて[Homebrew]をインストール。

```Bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

[Homebrew]: https://brew.sh/index_ja.html

### Git

バージョンが新しいとなんとなく嬉しいので導入。
ちゃんと新しくなってるな、よしよし。

```Bash
$ brew install git

$ git --version
git version 2.12.0
```

### fish shell

Fish使いなので導入
ついでにログインシェルを書き換える

```Bash
$ brew install fish

$ which fish
/usr/local/bin/fish

$ sudo echo "/usr/local/bin/fish" >> /etc

$ chsh -s /usr/local/bin/fish
```

永続化メモ

```Bash
# PATHの永続化
$ set -U fish_user_paths $HOME/.nodebrew/current/bin $fish_user_paths

# 環境変数の永続化
$ set -xU NODE_PATH $HOME/.nodebrw/current/lib/node_modules

# aliasの永続化
$ alias dc docker-compose
$ funcsave dc
```

参考リンク:

- [【Mac】ログインシェルの変更方法について (chsh, chpass)](http://www.task-notes.com/entry/20150117/1421482066)
- [funcsave:【要注意】関数定義をファイルに保存する](http://fish.rubikitch.com/funcsave/)

### Node

[Nodebrew]を使ってインストール
NODE_PATHが公式に書いてない罠さえ除けば最強だ
最近stableとlatestの違いがわからない件について

参考リンク:

- [Nodebrew]
- [node.js のrequire先pathにnpm -gでグローバルインストールしたのものを登録する - Qiita](http://qiita.com/hikaruna/items/abdadca27f12c0e4eb78)

```Bash
$ curl -L git.io/nodebrew | perl - setup

# fish用のセットアップ
$ set -U fish_user_paths $HOME/.nodebrew/current/bin $fish_user_paths

# このままだとグローバルにインストールしたパッケージが使えないので設定
$ set -xU NODE_PATH $HOME/.nodebrw/current/lib/node_modules

$ nodebrew instal-binary latest

$ nodebrew use latest

$ node -v
v7.2.1

$ npm -v
3.10.10

# いつものAltJSも入れた
$ npm install -g livescript prelude-ls
```

[Nodebrew]: https://github.com/hokaccha/nodebrew

### Docker

折角のMacbookProなんだからDocker入れておかないと損だ。

参考リンク:

- [Docker for Mac](https://docs.docker.com/docker-for-mac/install/)
- [Docker Compose](https://docs.docker.com/compose/install/)

```Bash
$ docker version
Client:
 Version:      17.03.0-ce
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   60ccb22
 Built:        Thu Feb 23 10:40:59 2017
 OS/Arch:      darwin/amd64

Server:
 Version:      17.03.0-ce
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 07:52:04 2017
 OS/Arch:      linux/amd64
 Experimental: true

# ついでにdocker-composeも導入する
$ curl -L "https://github.com/docker/compose/releases/download/1.11.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ chmod +x /usr/local/bin/docker-compose

$ docker-compose --version
docker-compose version 1.11.2, build dfed245
```

### Vim

メイン獲物
デフォルトのVimはレジスタの*がクリップボードとリンクしていないのでMacvimで置き換える

```Bash
$ brew install macvim --with-override-system-vim

$ vim
:PlugInstall

# Ag に依存しているのでインストール
$ brew install ag
```

## 設定

SierraにしたのでKarabinerはまだ導入できない。
幸いすでにターミナルでほぼ全ての作業が完結するので、キーリピートしか使ってなかったしちょうどいいや。

- キーリピートの高速化
  - [Macのキーリピートをシステム設定での設定よりも速くする](http://drumsoft.com/blog/archives/21)
    ```Bash
      $ defaults write NSGlobalDomain KeyRepeat -int 1
      $ defaults write NSGlobalDomain InitialKeyRepeat -int 10
    ```
- [dotfiles](https://github.com/miyabisun/dotfiles)
  - tmuxとvimの設定はここに逃していたので一撃で導入完了
    ```Bash
      curl -L https://raw.github.com/miyabisun/dotfiles/master/install | bash
    ```

## 環境構築

### ブログ (Hexo)

[Hexo]のDisqus設定がテーマ別なのでGitからクローンしただけだと引っ越しと同時に吹っ飛んでしまう。
lightテーマの人のGitHubをフォークしてきて、それを更新することで対応。
……こういう風に使うもんだっけ？

[Hexo]: https://hexo.io/

