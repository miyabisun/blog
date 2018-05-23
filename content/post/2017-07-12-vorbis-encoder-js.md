---
title: vorbis-encoder-jsをリリース
date: 2017-07-12T09:56:54+09:00
tags:
  - Ogg
  - JavaScript
  - Emscripten
id: ogg
---

[Browserify] を利用する [vorbis-encoder-js] をリリースしたので、
履歴とハマリポイント等を備忘録として書き残していく。

<!-- more -->

## 既存のライブラリでは何が足りなかったのか

- [higuma/ogg-vorbis-encoder-js - GitHub](https://github.com/higuma/ogg-vorbis-encoder-js)
- [Garciat/libvorbis.js - GitHub](https://github.com/Garciat/libvorbis.js)

上記どちらのライブラリでも [AudioBuffer] から [Ogg Vorbis] 形式の音楽ファイルを生成し、DLさせる所までは実現した。
しかし、元々想定される用途が [Web Audio API] を利用して、マイクからの音声データをDLする所がゴールだったようで、
タグ情報を埋め込む事考えられておらず、任意のタグ情報が埋め込めない。

また、モダンなプロジェクトなら [Browserify] や [webpack] の事は考慮に入れると思うのだが、そうでもないらしく微妙に使い勝手が悪い。
最初のコミットが1年も2年も前なので仕方ない側面もある。

そういった経緯から要望を満たすライブラリが無かったので、
GitHub で公開されている既存プロジェクトのソースコードを参考に作る事にした。

## 要件

- Node.js での利用を想定する
- JavaScript での利用は [Browserify] を利用する
- 任意のタグ情報を設定出来ること
- ビルド環境の構築は頑張らない

## ログ

### libvorbis のビルド

[libvorbis] はC言語で書かれているので、JavaScriptに変換するために [Emscripten] を経由する必要がある。
もともとC言語で書かれたライブラリを任意の言語に変換して使おうという要望は [LLVM] というプロジェクトで実現されており、
[Emscripten] は [LLVM] が生成した中間コードを JavaScript に変換する機能に特化している。

従って、あれやらそれやらのインストールが必要で、
いつまで経ってもコンパイルが通る環境が作れない！

参考リンク：[emscriptenで遊んでみた - kashiの日記](http://verifiedby.me/adiary/093)

結局 [Emscripten] に特化したコンテナは Docker Hub で公開されていたので利用することにした。

参考リンク：[apiaryio/emcc - Docker Hub](https://hub.docker.com/r/apiaryio/emcc/)

### Makefile の作成

ベースとして利用させて頂いている [higuma/ogg-vorbis-encoder-js - GitHub](https://github.com/higuma/ogg-vorbis-encoder-js) のプロジェクトでは、
作者さんが Ruby が好きなのか Rake でソースコードのビルドを行っている。
2年前のプロジェクトなので、 Grunt や Gulp 等の Node.js 製タスクランナーは馴染みが無かったと思われる。

折角なんで Gulp で書いても良かったが、勉強も兼ねて Makefile に移植することにした。
[Emscripten] は C言語なんでそっちの方が良さそうだ。
結果約2週間ほどかかった。

ハマった主な要因としては、function や foreach を使う手法はバッドノウハウになりがちである事に気が付かなかった。
Makefile の本質はビルド後のファイルを作成することにあるので、素直に依存ファイルを並べていく方が良い事に気がついてからはすんなり実装が完了した。

参考リンク：

- [GNU make](https://www.gnu.org/software/make/manual/html_node/index.html#SEC_Contents)
- [8.5 The foreach Function - GNU make](https://www.gnu.org/software/make/manual/html_node/Foreach-Function.html)
- [function define in makefile - Stack Overflow](https://stackoverflow.com/questions/21324073/function-define-in-makefile)
- [Makefile の書き方 (C 言語) - WTOPIA v1.0 documentation](http://www.ie.u-ryukyu.ac.jp/~e085739/c.makefile.tuts.html)
- [Emscripten Compiler Frontend (emcc)](https://kripken.github.io/emscripten-site/docs/tools_reference/emcc.html)

### encoder.js の作成

[Emscripten] では生成した JavaScript ファイルに `Module` という変数を生成して関数一覧を用意し、
module.exports に代入してくれるんで、普通に Node.js のやり方が通用する。

ベースのプロジェクトではpre.jsやpost.jsを利用して、
libvorbisに直接関数を足して対応するという形であったので、普通にrequireして使うように調整。

### タグの埋め込み

後は消化試合、テストファイルでも作って終わらせばOK…になるわけが無かった。
[Emscripten] で出力した関数が文字列を解釈できず大幅に足止め。
結局これも2週間かかった。

[emscriptenでC/C++プログラムをwebブラウザから使うまでの難所攻略](https://www.slideshare.net/llamerada-jp/cmu29?next_slideshow=1)で紹介されているスライドが勉強になった。

> - emscriptenでは、C/C++の変数をArrayBufferで作った擬似的なメモリ空間に保存している
> - Module.HEAPU8などDataViewでアクセスできる

これのお陰で、[Emscripten] は単なるトランスパイラではなく、
C言語そのものを JavaScript で表現しているのだという事が理解できた。
その視点でドキュメントやら紹介記事を読み返した事で解決。

参考リンク：

- [emscriptenでC/C++プログラムをwebブラウザから使うまでの難所攻略 - SlideShare](https://www.slideshare.net/llamerada-jp/cmu29?next_slideshow=1)
- [Emscriptenで、C++からJavaScriptの関数を呼び出す - Gist](https://gist.github.com/faithandbrave/0362f25bc355d529ab1c)
- [EmscriptenでC言語をJavaScriptに変換する - Qiita](http://qiita.com/sassy_watson/items/3ec69b19a22a457362a9)
- [preamble.js - Emscripten](https://kripken.github.io/emscripten-site/docs/api_reference/preamble.js.html)
- [Emscripten C++/OpenGL ES 2.0 (9) C関数呼び出しと FileSystem - ホイール欲しい ハンドル欲しい](http://wlog.flatlib.jp/item/1709)

### テスト

いつも通り Mocha + Chai + LiveScript というテンプレ構成で実装。
でもNode.jsはAudioBufferないじゃん、テスト作れないじゃんというわけで調査。

#### 依存ライブラリ

普通にnpmを探したら置いてあった。
[audio-decode](https://www.npmjs.com/package/audio-decode)を組み込んで実装。
しっかりAudioBufferオブジェクトが帰ってくる。

テスト音楽ファイルは[audio-lena](https://www.npmjs.com/package/audio-lena)が最適。
ええ声のお姉さんが「お〜れいるひ〜」と12秒間アカペラで歌ってくれた。

参考リンク：

- [audio-buffer - npm](https://www.npmjs.com/package/audio-buffer)
- [audio-decode - npm](https://www.npmjs.com/package/audio-decode)
- [audio-lena - npm](https://www.npmjs.com/package/audio-lena)

#### テストの実装

実際の実装はすんなり行かず、大体3日程かかった。
Node.js には Blob が存在しないので Buffer を使って上手くやるようにした。

Node.js のドキュメントをみたところ、
Buffer.concat の項目によれば Uint8Array が詰まった配列でもいいらしい。
楽勝だな。

> list <Array> List of Buffer or Uint8Array instances to concat

エラーが出まくって結合出来ない。
大嘘じゃねーか！…一応History見ておくか。

> History:
> v8.0.0: The elements of list can now be Uint8Arrays.
> v0.7.11: Added in: v0.7.11

なるほど、Uint8Array許可はNode.jsのv8からなのね。
一旦は下記で対応。

```JavaScript
Buffer.concat(
  this.ogg_buffers.map(function(it){
    return new Buffer(it.buffer);
  })
);
```

参考リンク：

- [proxyquire](https://github.com/thlorenz/proxyquire)
- [Node v0.6.12 における Buffer と ArrayBuffer の違い - Null.ly](http://null.ly/post/18721753351/node-v0612-%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B-buffer-%E3%81%A8-arraybuffer-%E3%81%AE%E9%81%95%E3%81%84)
- [ArrayBufferの分割/結合方法 - Qiita](http://qiita.com/hbjpn/items/dc4fbb925987d284f491)
- [Class Method: Buffer.concat - Node.js](https://nodejs.org/api/buffer.html#buffer_class_method_buffer_concat_list_totallength)

### 感想

最初から簡単だとは思わなかったけど相当ハマった。
しかし、時間を掛けた甲斐はあったと思う。

仕事を辞めてPSO2で遊んでいるうちに、ネイティブで動作する言語に対する興味がどんどん湧いてきた。
今回の件でネイティブとWebの技術の橋渡しに関してイメージがかなり掴めたので、
もっと良いアイデアや実装ができそうな気がする。

興味が湧いたらIoTなんかにも手を出してみたい。

[vorbis-encoder-js]:https://github.com/miyabisun/vorbis-encoder-js
[Browserify]:http://browserify.org/
[webpack]:https://webpack.js.org/
[Ogg Vorbis]:http://www.vorbis.com/
[AudioBuffer]:https://developer.mozilla.org/ja/docs/Web/API/AudioBuffer
[Web Audio API]:https://developer.mozilla.org/ja/docs/Web/API/Web_Audio_API
[libvorbis]:https://git.xiph.org/?p=vorbis.git
[Emscripten]:https://kripken.github.io/emscripten-site/index.html
[LLVM]:https://ja.wikipedia.org/wiki/LLVM

