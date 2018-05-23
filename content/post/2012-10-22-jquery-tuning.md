---
title: 【jQuery】セレクターのチューニング
date: 2012-10-22T14:43:33+09:00
tags:
  - JavaScript
id: jquery-tuning
---

## 結論

### jQueryのセレクタは右側から評価する

jQueryは長いセレクタの場合、右側から解釈するので、
ヒット数の多いものが右側に来ると非常に時間が掛かる。

```JavaScript
// NG: li要素の一覧を先に取得するので、IDで絞った意味がない
$("#conditions li");

// OK: このように使えばIDがconditionsのものから検索出来る
$("#conditions").find("li");
```

<!-- more -->

### 重い処理はキャッシュを使う

変数を宣言して、jQueryオブジェクトを保存することをキャッシュと呼ぶらしい。
例えば何度も`hoge`クラスの要素を使う場合、
下記のように書いておけばHTMLを全て検索する処理は一度で済む。

```JavaScript
var $hoge = $(".hoge");
var $piko = $hoge.find(".piko");
```

## 速度改善のきっかけ

jQueryを使用しているページを業務内で触っていたが、
Tableの表示がものすごく遅い…という訳で
CSSやJavascriptを修正して速度改善することになった。

### 条件
- Excelの「ウィンドウ枠の固定」っぽい表示にする
  - jQueryのライブラリを使用して実装すること
- カラムの動的な表示・非表示を行う
  - チェックボックスを複数作成し、チェックが消えるとカラム非表示

## ぐぐってみた

### JavaScriptコーディング等を書く上でのパフォーマンス確認事項３０
<http://phpspot.org/blog/archives/2010/06/javascript_87.html>

JavaScriptに関しては良い感じだが、jQueryに関しては少ない。
幾つかの項目は可読性が犠牲になるので余裕があればで良いだろう。

### jQuery("ul > li") のように使う。 jQuery("ul li") は広義すぎる

jQueryは右側から解釈するのでどちらもNG
例ならjQuery("ul").children("li")、もしくはjQuery("ul").find("li")とするのが良いだろう。

### jQueryを良くする25のTIPS
<http://blog.webcreativepark.net/2008/12/17-225630.html>

最初に読む書籍としてはかなり良い。
口語が多いので十分理解できたら別のページがよさそう。

### jqueryのセレクタを高速化する
<http://nex.xrea.jp/?s=1231>
シンプルな結論だった

