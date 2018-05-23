---
title: 【ConoHa】APIからデフォルトのイメージ一覧を取得する
date: 2016-02-04T09:13:46+09:00
tags:
  - ConoHa
id: conoha-api-images
---

※本投稿は調査段階のメモ書きです

## 何処で取れるのか？

イメージ一覧取得(nova) - Compute Service
<https://www.conoha.jp/docs/compute-get_images_list.html>

<!-- more -->

### ログインしてトークンを受け取っていないと使えない

ログインしなくてもアクセス出来そうな情報ではあるのだが…

```Bash
$ curl https://compute.tyo1.conoha.io/v2/1864e71d2deb46f6b47526b69c65a45d/images
Authentication required
```

## 今やろうとしてること

仕事であちこちのブランチをちょこちょこ弄るのに苦戦中。
これを解決するために、ConoHaのインスタンスを複数作って臨機応変に対応出来るようにしたい。

### 利用予定の技術

これらを駆使してインスタンスを好きな時に生やせる仕組みを作る

- BitBucket
- ConoHa API
- Vagrant
- Ansible

### 進捗

VagrantでConoHaのインスタンスを作成する際、正確なプラン名、イメージ名が必要になる。
しかし、公式のAPIを眺めても理解出来なかったので、結局ググッて他のサイトからどのAPIが該当するのか調べる事になった。

というのも、イメージと一言に言っても今回の目的であるテンプレートのイメージの他に、ユーザーが登録したISOイメージ、スナップショット等がありどれが対象かは一目で分かりづらい。

最終的にはコマンドラインツール化等もしていきたいので、手早くしないとね…
