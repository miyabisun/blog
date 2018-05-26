---
title: Nim Compiler vol.1
date: 2018-05-25T09:26:10+09:00
tags:
    - Nim
    - Docker
draft: false
---

現在Nimのクロスコンパイラ環境をDockerで作ろうとしている。  
Chromeのタブを開きまくっているので一度整理したいと思って記述

<!--more-->

## 目的

`echo "hello nim"`1行を作ったしょぼいファイルを作成した。  
`src/hello.nim`として保存して、各OSで動作する仕組みを構築したい。

## Linux -> Windows

LinuxからWindowsへクロスコンパイルする仕組みを検討。  
まずはQiita等で情報収集をしていると、どうやらMinGWを使えば良いとのこと。

MinGWってWindowsのUnixライクなターミナルエミュレータ的なソフトじゃなかったっけ？  
[Wikipedia](https://ja.wikipedia.org/wiki/MinGW)によると「Minimalist GNU for Windows」の略らしい。  
シェルはMSYS、MinGWはUnix用ソフトウェアをWindowsで動かす為のプロジェクトだったのね。

### Dockerfile

DockerHubにあるNimのイメージバージョンによってディストリビューションをコロコロ換えてて、  
0.18現在ではAlpineしか用意されていなかった。  
なのでAlpineにMinGWを導入したものを探した所、[portown/alpine-mingw-w64](https://hub.docker.com/r/portown/alpine-mingw-w64/~/dockerfile/)がヒット。

[nimlang/nim](https://hub.docker.com/r/nimlang/nim/tags/)をベースに上記Dockerfile記述のものを取り入れようとしたがDL→インストールがうまく動作せずに断念。  
調査した所、Dockerfileでのビルド時に「multi stage build」という手法が使えるらしい。  
`src/hello.nim.cfg`を下記のように修正して

```Nim
amd64.windows.gcc.path = "/usr/local/mingw/bin/"
amd64.windows.gcc.exe = "x86_64-w64-mingw32-gcc"
amd64.windows.gcc.linkerexe = "x86_64-w64-mingw32-gcc"
amd64.windows.gcc.options.linker = ""
```

コマンドを実行すると、`src/hello.exe`ファイルが生成された。  
(なお動作は未確認)

```Bash
$ nim c --os:windows --cpu:amd64 -d:release src/hello.nim
```

### 参考サイト

- 全体的な調査
    - [Nim Compiler User Guide](https://nim-lang.org/docs/nimc.html)
- Dockerfile
    - [複数の言語の runtime を持つ Docker image を作る - Qiita](https://qiita.com/izumin5210/items/cf14a1ea58fb82d36bb2)
    - [Docker multi stage buildで変わるDockerfileの常識 - Qiita](https://qiita.com/minamijoyo/items/711704e85b45ff5d6405)
    - [nimlang/nim - Docker Hub](https://hub.docker.com/r/nimlang/nim/tags/)
- Linux -> Windows
    - [Nimクロスコンパイルのやり方(Linux->Windows版) - Qiita](https://qiita.com/6in/items/c72d73d063146e6272fa)
    - [MinGW - Wikipedia](https://ja.wikipedia.org/wiki/MinGW)
    - [EASY WINDOWS AND LINUX CROSS-COMPILERS FOR MACOS](https://blog.filippo.io/easy-windows-and-linux-cross-compilers-for-macos/)
    - [portown/alpine-mingw-w64 - Docker Hub](https://hub.docker.com/r/portown/alpine-mingw-w64/~/dockerfile/)

## Alpine -> Ubuntu

Dockerで作ったコンテナがAlpine Linux  
コンパイル後にそのまま吐き出されたバイナリファイルを実行すると`hello nim`が表示されるが、  
Ubuntuのメインマシンに持ち帰って実行するとさっぱり動かない。

```Bash
# コンテナnimlang/nim内でコンパイル実行
$ nim c src/hello.nim

# Ubuntuに戻って実行
$ src/hello
Failed to execute process 'src/hello'. Reason:
The file 'src/hello' does not exist or could not be executed.
```

### 調査

Golangがワンバイナリ吐き出すので、それ関係で色々調べた所  
C言語では動的リンクと静的リンクという概念がある事がわかった。

早速「C スタティックリンク」というワードで検索。  
ファイルサイズが小さくなるように依存モジュールはOSのものを利用する想定でコンパイルするのが普通、  
可搬性を重視する時は依存モジュールをバイナリの中に含めてファイルサイズを大きくする。

### 実践

Nimでそれをやるのにはどうすれば良いか調べた所。  
この内容の`hello.nim.cfg`を`hello.nim`と同階層に設置。

```Nim
@if musl:
  passL = "-static"
  gcc.exe = "/usr/local/musl/bin/musl-gcc"
  gcc.linkerexe = "/usr/local/musl/bin/musl-gcc"
@end
```

更にコマンド実行時にオプションを追加

```Bash
$ nim c -d:musl -d:release src/hello.nim

# alpine
$ src/hello
hello nim

# ubuntu
$ src/hello
hello nim
```

エラーが出ずに動かなくなる事を確認できた

### 参考サイト

- [NimライブラリをReact Nativeから使いたい(調査編2)](https://takalog.hatenadiary.com/entry/2018/03/29/002442)
- [スタティックリンク(静的リンク)とダイナミックリンク(動的リンク) - 落穂拾い](http://www.cc.kyoto-su.ac.jp/~kbys/kiso/cpu/dynamic-link.html)
- [Static Linking With Nim](https://www.reddit.com/r/programming/comments/2wk7q6/static_linking_with_nim/)

## ToDo

動機がリンクを整理したかっただけで、まだまだ先は長いので引き続き調査＆動作検証を行っていく。

- Windows環境での動作チェック
- Mac用のコンパイラの導入
- 今作っているNim用のコマンドラインツールでの動作検証

### 参考サイト

- Linux to MacOS
    - [Nim > forum index > Cross compile to OS X](https://forum.nim-lang.org/t/2652)
    - [tpoechtrager/osxcross - GitHub](https://github.com/tpoechtrager/osxcross)
    - [Nimクロスコンパイルのやり方 - Qiita](https://qiita.com/hatchet/items/24569bb852fef41b8a62)

