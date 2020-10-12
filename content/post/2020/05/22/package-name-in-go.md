+++
title= "Goのパッケージ名は単数形？複数形？"
date= 2020-05-22T09:16:17+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["gotips","golang"]
keywords = ["Go","Go言語","パッケージ"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goのパッケージ名には複数形の名前はつけないほうがよいというメモ。

<!--more-->

# TL;DR
- 標準パッケージを参考に基本は単数形で命名する
- 特別な事情があるときだけ複数形の名前をつける

# Goのネーミング
Goのパッケージ名や変数名の名前付けにはいくつかの指針がまとめられている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://micnncim.com/post/2019/12/11/go-naming-conventions/" data-iframely-url="//cdn.iframe.ly/XeEUyQY?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Effective Goなどにもパッケージ名の節がある。

- https://golang.org/doc/effective_go.html#package-names
- https://github.com/golang/go/wiki/CodeReviewComments
- https://dave.cheney.net/2019/01/08/avoid-package-names-like-base-util-or-common

「短く簡潔なもの名前ほどよい」や「どのようにパッケージを分割するか」という指針はある。  
しかし「そもそも複数形を使うべきか」のような説明はないような気がするので一筆書いてみる。

# 原則単数形でパッケージ名を決めるべき
たまに`controllers`のようなパッケージ名を見たりするが、Goの場合は単数形のパッケージ名にするほうがよいと考える。  
根拠となるような言及された情報があるわけではないが、標準パッケージは原則単数形のパッケージ名になっている。

# stringsパッケージやerrorsパッケージは複数形では？
ご存知のとおり、標準パッケージにも例外的に複数形は存在する。
というかそこそこの数が存在していたりする。
以下のパッケージは多用されるので存在を知っている人も多いだろう。

- https://golang.org/pkg/strings/
- https://golang.org/pkg/bytes/
- https://golang.org/pkg/errors/
- https://golang.org/pkg/go/types/

これらは単純に予約語やプリミティブ型と同名になってしまうから複数形になっているだけだ。  

- https://golang.org/ref/spec#Keywords
- https://golang.org/ref/spec#Types

なので、予約語や既定の名前に衝突しない限り単数形のパッケージ名をつければよい。

# 終わりに
とはいえ、Go本体のパッケージにも`cmd/asm/internal/flags`のような複数形のパッケージはまれにある。

# 参考
- https://micnncim.com/post/2019/12/11/go-naming-conventions/
- https://golang.org/doc/effective_go.html#package-names
- https://github.com/golang/go/wiki/CodeReviewComments
- https://dave.cheney.net/2019/01/08/avoid-package-names-like-base-util-or-common

