---
title: 履歴書・職務経歴書
date: 2017-08-08T21:54:19+09:00
tags:
  - Node.js
id: job-transfer
---

再就職が決まった。

今回の就職活動に利用した
履歴書・職務経歴書をアップデートするのにGitに乗り換えた話。

<!--more-->

## 冒頭

再就職が決まった。
そろそろ就職考えないとなぁと[Forkwell jobs](https://jobs.forkwell.com/)というサービスになんとなく登録したところ、2日足らずでスカウトしていただいた。
最初はあまり乗り気では無かったのだが、必要とされてる感っていうのかな？
相手の企業のサイトやサービスを調べている内に情が出てきて、結果トントン拍子で採用が決まったのだ。

採用のやり取りが進む中、履歴書・職務経歴書をPDFで送信することになった。
ハードディスクを漁って出てきたのは大昔に作った[WPS Office](https://www.kingsoft.jp/office/)の履歴書と職務経歴書

今更これメンテするのかよ。。。
それに、エンジニアとしてエクセル方眼紙はちと恥ずかしいので脱Officeしたいな。
じゃあ動的に印刷目的のHTMLファイルを作成して、それをPDFに加工して提出というのはどうだろう。

## 要件

- JSONファイルの情報をGitで管理出来るようにしたい
- Pug, Stylus でHTMLファイルを作成する
- Express で出来栄えを確認したいな

## ログ

### 職務経歴書

まずは調査だ。
履歴書は住所やら電話番号がネット上に晒されちゃうのでGitHubでの公開は流石になし。
でも職務経歴書をアップしている人は多いみたいだね。

参考URL：

- [職務経歴書OSS化 - Qiita](http://qiita.com/okoysm/items/abcad0b4aefa585bc50b)
- [職務経歴書をGitHubに公開するのはいいぞ #職務経歴書 - 空飛ぶ羊](http://okoysm.hatenablog.jp/entry/2016/12/19/060000)
- [okoysm/Curriculum-Vitae-template - GitHub](https://github.com/okoysm/Curriculum-Vitae-template)

調査内容に従って自分もGitHubにリポジトリを作成してアップロードしてみた。
様式は上記のテンプレートをお借りして、それに乗っかる形で作成。
[miyabisun/Curriculum-Vitae - GitHub](https://github.com/miyabisun/Curriculum-Vitae)

リンクにあるように、`github.com`のドメイン名を`gitprint.com`に変更すると、
GitHub上に上げたMarkdownファイルをPDFとして閲覧したりダウンロードできるらしい。
へーこれは便利なサードパーティー！？作った人のイケメンぶりに惚れた。

### 履歴書

最新版のChromeはデフォルトで`Ctrl + P`のショートカットキーから、
ダイレクトにPDFに保存する機能が備わっている。
おっ、これはPDFで提出しやすそうだな。

HTMLベースならGitでの世代管理もし易いし採用。
もちろん生のHTMLを触るとかありえないのでPugとStylusを使って。。。
ついでにちょこちょこ生成して確認したいからExpressでローカルサーバー立ち上げて、リロードで確認出来るようにした。

完了して提出、テンプレート化してGitHubにも公開した。
[miyabisun/job-transfer - GitHub](https://github.com/miyabisun/job-transfer)

まぁ、流石に個人情報全てをGitHubで公開するのは怖いので、
クリティカルなデータはbitbucketのプライベートリポジトリに叩き込んで、そこからシンボリックリンクを張って出力するように修正。

実装は履歴書・内定承諾書の両方を作るのに10時間弱だった。
そこからテンプレート化してGitHubに登録するのに2時間。
楽しめて作れたし、しっかり仕様を考えながらの作業にしては短時間で作れたんじゃなかろうか？

### 今後やりたい内容

HTML -> PDFを直接作りたい。
でも、それにはNode.js単体では実伝することは困難なようだ。
ざっとnpmを調査したところ

- [html-pdf](https://www.npmjs.com/package/html-pdf): 要phantomjs
- [html-to-pdf](https://www.npmjs.com/package/html-to-pdf): 要Java SDK
- [html-to-pdf-conversion](https://www.npmjs.com/package/html-to-pdf-conversion): 要[pdflayer](https://pdflayer.com/) API
- [phantom-html-to-pdf](https://www.npmjs.com/package/phantom-html-to-pdf): 要phantomjs

この中ではphantomjsが一番楽か？
しかし、node-gypがWindowsだとコケまくってハマるんだよなぁ…Docker使ってなんとかならんか？
