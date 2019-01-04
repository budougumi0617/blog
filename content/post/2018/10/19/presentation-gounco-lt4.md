+++
title = "[発表資料]go-cloudとWireを利用したDI #gounco #go"
date = 2018-10-19T13:31:21+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["presentation", "go"]
tags = ["wire", "golang","go-cloud"]
twitterImage = "logos/Go-Logo_Aqua.png"
keywords = ["Go", "wire", "dependency injection"]
+++

Go(Un)Conference（Goあんこ）LT大会 4kgの発表資料と資料中の参考リンク、補足をまとめた。

<!--more-->

|イベント名|Go(Un)Conference（Goあんこ）LT大会 4kg|
|---|---|
|URL|https://gounconference.connpass.com/event/99487/|
|会場|株式会社アイスタイル 東京都港区赤坂1-12-32(アーク森ビル34F)|
|日時|2018/10/19(金) 19:30 〜 22:00|
|ハッシュタグ| [#gounco](https://twitter.com/hashtag/gounco)|


---

- https://speakerdeck.com/budougumi0617/go-cloud-and-dependency-injection-by-wire

<script async class="speakerdeck-embed" data-id="dfd2df0a6ebe415193e484b31a0156f9" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# go-cloud
go-cloudはGoogleが公開したポータブルなクラウドアプリケーションを開発するためのGo用のAPIライブラリ。  
2018/07/24にGo blogで"Portable Cloud Programming with Go Cloud"というタイトルでリリースがアナウンスされた。

- Portable Cloud Programming with Go Cloud
  - https://blog.golang.org/go-cloud

# Wire
Wireコマンドはgo-cloudリポジトリに付属されているコマンドラインツール。


https://github.com/google/go-cloud/tree/master/wire

インストールは以下の`PATH`で`go get`できる。

```bash
$ go get github.com/google/go-cloud/wire/cmd/wire
```

# サンプルコードについて
発表中のサンプルコードはgo-cloudリポジトリに含まれているサンプルコードを基にしている。

- https://github.com/google/go-cloud/tree/master/samples/guestbook

# パッケージとしての依存性について
wireコマンドを使ってもパッケージとしての依存性は切り離せない。  
例えば「GCP向けのビルドをするときはGCP以外への依存をバイナリに含めたくない」というときは一工夫が必要になる。  
詳細は以下の記事を参照のこと。

- [go listで依存パッケージを一覧する。-tagsで依存パッケージを切り替える #go](/2018/09/21/package-dependencies-with-go-list-and-build-tags/)


# 関連
- [当ブログのGo記事一覧](/categories/go/)
- [go listで依存パッケージを一覧する。-tagsで依存パッケージを切り替える #go](/2018/09/21/package-dependencies-with-go-list-and-build-tags/)


