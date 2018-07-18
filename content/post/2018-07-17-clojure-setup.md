---
title: "Clojure Setup (Spacemacs編)"
date: 2018-07-17T15:52:48+09:00
tags:
    - Clojure
    - Emacs
    - Vim
---

## 概要

Clojureを扱っている企業へ就職した。  
元々Vimを使っていたが、ClojureはLISP系ということでEMACSが職場内では一番多く使われているようだ。  
(というかメインVim使いは1人も居ない)

しかしデフォルトのEmacsはVimと同じで中々使い辛いらしくプラグインやらをあれこれ導入したりと大変とのことで。  
[Spacemacs](https://github.com/syl20bnr/spacemacs)というEmacsのディストリビューション(?)が存在し、  
それの導入方法を教わったので備忘録としてまとめる。

因みにSpacemacsはインストール後の最初の起動で、VimとEmacsのどちらのモードを使いますか？という選択肢が表示される。  
完全なVimとは言い難いのだが、Vimライクな操作が可能なのである程度簡単に乗り換える事が出来る。  
当面の間はClojure系はSpacemacs、その他はVimを使って行こうかなと考えている。

## インストール方法(MacOS)

一度も起動していなければ`~/.emacs.d`や`~/.emacs`等のファイルは作成されないらしいので、  
HomebrewでEmacsをインストール後、そのままGitHubからSpacemacsを落としてくれば使えるようだ。

```bash
$ brew tap d12frosted/emacs-plus
$ brew install emacs-plus
$ emacs --version

$ git clone https://github.com/syl20bnr/spacemacs ~/.emacs.d
```

## 起動方法

今回導入した`emacs-plus`はGUIアプリケーションのようで、  
ターミナルに覆いかぶさるようにSpacemacsが立ち上がる。

非常に邪魔なのでコマンドラインで動作させようと思い調査、  
`-nw`オプションを指定することでCLI版として動作する。

```bash
$ emacs --help
Usage: /usr/local/Cellar/emacs-plus/26.1/Emacs.app/Contents/MacOS/Emacs [OPTION-OR-FILENAME]...
# 省略
--no-window-system, -nw     do not communicate with X, ignoring $DISPLAY

# CLI版で立ち上がる事を確認
$ emacs -nw

# fishでエイリアスを張る
$ em
fish: Unknown command 'em'
$ alias em='emacs -nw'
$ funcsave em

# 張ったエイリアスで立ち上がる事を確認
$ em
```

## Stable版パッケージを導入する

SpacemacsはSNAPSHOTのβ版ライブラリも積極的に導入するらしい。  
それによりSpacemacsが動作しなくなる現象が多数報告されており、入社1日目で早速洗礼を受けてしまった。

内容はCider0.18.0SNAPSHOTによりcider-refreshショートカットが突如削除されてなくなり、  
代わりにcider-ns-refreshショートカットに変更されているというもの。  
Spacemacsはその変更についていけず、`Space > m > s > x`で`cider-refresh`を実行しようとしてショートカットが存在しませんエラー。

これはたまらないということで、Stable版を導入するよう設定を変更することで対処。

```Bash
# まずはSpacemacsの起動で変更したファイルやライブラリを全て初期化
$ cd ~/.emacs.d/
$ git clean -fdx
Skipping repository .cache/quelpa/melpa
Removing .cache/quelpa/cache
Removing .cache/quelpa/build
Removing .cache/savehist
Removing .cache/places
Removing .cache/projectile-bookmarks.eld
Removing .cache/layouts
Removing .cache/recentf
Removing .cache/spacemacs-buffer.el
Removing .cache/auto-save
Removing elpa/

# ホームディレクトリの.spacemacsを編集
$ vim ~/.spacemacs
# dotspacemacs/user-init 関数内に以下の2行を追加
(defun dotspacemacs/user-init ()
  "Initialization function for user code.
It is called immediately after `dotspacemacs/init', before layer configuration
executes.
 This function is mostly useful for variables that need to be set
before packages are loaded. If you are unsure, you should try in setting them in
`dotspacemacs/user-config' first."
  (push '("melpa-stable" . "stable.melpa.org/packages/") configuration-layer--elpa-archives)
  (push '(use-package . "melpa-stable") package-pinned-packages)
  )

# 再起動でspacemacsのライブラリのダウンロードが開始される(100個くらいあるのでパケ死しないよう注意)
$ emacs -nw
```

## 頻出ショートカット

- Space -> Space: Emacsコマンド検索&実行
- Space -> f -> t: ファイルツリー
- Space -> b -> b: buffer切り替え
- Space -> m -> s -> i: cider-jack-in
- Space -> m -> s -> s: REPL起動
- Space -> m -> s -> x: リフレッシュ

開いたファイルはbuffer切り替えで変更していくようだ。  
なんかSublimeTextのCtrl+Pを思い出す動作。

## Links

- [syl20bnr/spacemacs - GitHub](https://github.com/syl20bnr/spacemacs)
- [公式のインストール](https://github.com/syl20bnr/spacemacs#install)
- [Document](http://spacemacs.org/doc/DOCUMENTATION.html)
- [spacemacsについて ~~spacemacsが最強で最高で神エディタであると思い始めた件~~](https://qiita.com/nobkz/items/994c6b3e6f42a0e33fef)
- [Spacemacsでstableのpackageを使うように指定する](https://scrapbox.io/makinoshi/Spacemacs%E3%81%A7stable%E3%81%AEpackage%E3%82%92%E4%BD%BF%E3%81%86%E3%82%88%E3%81%86%E3%81%AB%E6%8C%87%E5%AE%9A%E3%81%99%E3%82%8B)
- [Spacemacsとやらをインストールしてみる - Qiita](https://qiita.com/clutter/items/1f60bdabb31dec9afe5d)
- [Spacemacsでstableのpackageを使うように指定する - makinoshi](https://scrapbox.io/makinoshi/Spacemacs%E3%81%A7stable%E3%81%AEpackage%E3%82%92%E4%BD%BF%E3%81%86%E3%82%88%E3%81%86%E3%81%AB%E6%8C%87%E5%AE%9A%E3%81%99%E3%82%8B)
- [spacemacs が起動しなくなった場合の回避策 - でらうま倶楽部](http://blog.livedoor.jp/tek_nishi/archives/9715298.html)
