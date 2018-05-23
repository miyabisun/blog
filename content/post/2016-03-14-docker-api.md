---
title: Docker APIでログインする
date: 2016-03-14 21:14:53
tags:
  - Docker
id: docker-hub
---

## Login

[Docker Tutorial Series, Part 7: Ultimate Guide for Docker APIs](http://blog.flux7.com/blogs/docker/docker-tutorial-series-part-7-ultimate-guide-for-docker-apis)

```Bash
$ curl -L -u userid:password https://index.docker.io/v1/users
"OK"

$ curl -L -u userid:bad-password https://index.docker.io/v1/users
""
```

<!-- more -->

## Node.jsで書いてみる

[node.jsでbasic認証を使ってhttps接続する - @blog.justoneplanet.info](http://blog.justoneplanet.info/2011/09/30/node-js%E3%81%A7basic%E8%AA%8D%E8%A8%BC%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6https%E6%8E%A5%E7%B6%9A%E3%81%99%E3%82%8B/)
どれどれ…結構簡単に実装できそうだな。
利用言語はいつも通りLiveScript

```test.ls
require! \https

options =
  host: \index.docker.io
  path: \/v1/users/
  auth: "#{process.argv.2}:#{process.argv.3}"

https.get options, (res)->
  console.log res.status-code
  res.on \data, -> console.log it.to-string!
```

```Bash
$ lsc test.ls userid password
200
"OK"

$ lsc test.ls userid bad-password
401
""
```

途中httpパッケージで必死にhttps通信しようとしてハマったりしたが、
httpsパッケージなるものがあることがわかってからは一瞬で実装出来た。
