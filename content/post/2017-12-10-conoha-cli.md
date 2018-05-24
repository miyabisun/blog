---
title: ConoHa CLI 開発日誌 vol.6
date: 2017-12-14T9:26:00+09:00
tags:
    - Golang
    - ConoHa
id: conoha-cli
---

ConoHa-CLIの進捗報告。
今回も前回と同様に仕様回りを固める

## 現状

- spec.tomlのフォーマットに関して
- instance.tomlは必要ないのではないかと考え直す
- conoha.tomlのフォーマットに関して

<!-- more -->

## spec.tomlのフォーマットに関して

[VM追加 - Compute API v2](https://www.conoha.jp/docs/compute-create_vm.html)を元に最低限必要なものを見繕う

- instance_name_tag : インスタンス名
- image: イメージ(UUID)
- flavor: VMプラン(UUID)
- user_data: cloud-config (URL or ./cloud.cfg)
- key_name: SSHキー名

おっ、最小で考えたら意外と少なくいけるな。
ユーザースクリプトがAPI越しだとcloud-configオンリーなのが悲しい。
ところで、考えないようにしてたけどこのイメージとVMプランのUUIDとはなんぞや？

イメージ一覧画面を見たが何も書いて無かったので、
[VM追加 - Compute API v2](https://www.conoha.jp/docs/compute-create_vm.html)のページから推測しよう。

> Request Json (最低指定時)

```JSON
{
    "server": {
        "imageRef": "1f7bcc63-4a18-4371-85b1-bcdd4301ff31",
        "flavorRef": "b60acd11-3fd5-46e1-9387-aae4737d49aa",
        "adminPass":"72LY2hf38Kf84vCy4sUr"
    }
}
```

ハイフンでセパレートされた`4-2-2-2-6バイトの文字列`がそれっぽいな。
どれどれ、この情報を踏まえてイメージ一覧を。。
[イメージ一覧取得(nova) - Compute API v2](https://www.conoha.jp/docs/compute-get_images_list.html)
images[n].nameの他にimages[n].idがあって、このidがUUIDの形式に一致することがわかる。

```JSON
{
    "images": [
        {
            "id": "fb1d084f-357f-40e2-a2c3-59b8ecc1f6f2",
            "links": [{link}, {link}, {link}],
            "name": "vmi-centos-7.1-x86_64"
        },
        ...
    ]
}
```

同じくプラン一覧の仕様も確認しておく。
マスクは掛けられているもののイメージ一覧と同様にidを確認すれば良さそうだ。
[VMプラン一覧取得 - Compute API v2](https://www.conoha.jp/docs/compute-get_flavors_list.html)

```JSON
{
    "flavors": [
        {
            "id": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
            "links": [{link}, {link}],
            "name": "2gb-flavor"
        },
        ...
    ]
}
```

## instance.tomlは必要ないのではないかと考え直す

Vagrantfileの弱点として、複数環境から1個のインスタンスを触るのが弱いという弱点があるように感じられる。
毎回APIに現状を問い合わせた方が速度は落ちるが確実っぽい。

ここはconoha.toml, spec.toml, cloud-config(名称未定)の3つでいくかな

## conoha.tomlのフォーマットに関して

上2つは実装済
まずは必須要件を考えて下記を考えた

まぁ、最初はdefault値は無くて手作業で作ればいいね。
なので作らなければならないのはssh_key_listのみ。

- auth: 認証情報
- token: トークンID、有効期限は1日なので定期的に取り直す
- ssh_key_list: 登録済みのSSHのキー一覧
- default:
- default/image: デフォルトで利用するイメージ(name, id)
- default/flavor: デフォルトで利用するプラン(name, id)
- default/ssh: デフォルトで利用するSSHキー(ConoHa登録名)

