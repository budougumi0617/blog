+++
title= "Golangtokyo25 Read Again Awesome Go Article"
date= 2019-06-19T23:02:01+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang"]
keywords = ["Go", "golang"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


golang.tokyo #25で過去の優良情報を振り返る発表を行なった。
この記事は発表の補足資料となる。

<!--more-->

|イベント名|golang.tokyo #25 - Go の郷に入る|
|---|---|
|URL|https://golangtokyo.connpass.com/event/133581/|
|会場|ウォンテッドリー株式会社 東京都港区白金台5-12-7 MG白金台ビル4F|
|日時|2019/06/18(火) 19:30 〜 22:00|
|ハッシュタグ|[#golangtokyo](https://twitter.com/hashtag/golangtokyo)|

当日利用したスライドは以下になる。

<script async class="speakerdeck-embed" data-id="9a992e8bf67843d595124ea0b1a8d2ff" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# Goの郷に入る前に
今回はgolang.tokyoメンバーがオススメする3年以上前に公開された記事・発売された書籍の優良情報をまとめた。
発表では各々の記事・書籍を紹介する前に、まず今回の発表の背景・Goの哲学について触れた。

## Goらしいとは
Goらしいとは一言で言うとSimplicity（簡潔性）だ。後述するRob Pike氏の「Simplicity is Complecated」と言う発表タイトルが一言で表している。
より詳細な解説については、私の言葉より@songmuさんの次の記事を読むといいだろう。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Femployment.en-japan.com%2Fengineerhub%2Fentry%2F2018%2F06%2F19%2F110000" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no" allow="autoplay; encrypted-media"></iframe>

また、このSimplicityが何を目的としているかはBrand bookのGoが達成したいMissionやValueを確認することでわかる。

- Go Brand Book
    - https://storage.googleapis.com/golang-assets/Go-brand-book-v1.9.5.pdf
- GoのMission
    - Creating software at scale
    - Running software at scale

開発とサービスのスケーラビリティを両立するために、Goの言語設計者らはSimplicityを答えとした。このSimplicityの概念を理解すればより「Goらしいコード」を書けそうだ。

## Go1.Xの言語仕様について
Go1.Xは1.0の頃から言語仕様はほとんど変わっていない。このGoの方針は初期のドキュメントで言及されている。

- Go 1 and the Future of Go Programs
    - https://golang.org/doc/go1compat

> It is intended that programs written to the Go 1 specification will continue to compile and run correctly, unchanged, over the lifetime of that specification. At some indefinite point, a Go 2 specification may arise, but until that time, Go programs that work today should continue to work even as future "point" releases of Go 1 arise (Go 1.1, Go 1.2, etc.).

Release Noteを確認すれば、ほとんど毎回言語仕様は変更されていないことがわかる。

- Release History
    - https://golang.org/doc/devel/release.html

言語仕様が更新される際はほぼ微調整レベルの変更で、予約語もずっと25個のままだ。
（どちらが良いかは別として、）他の言語では古い情報は「○○仕様が増えたから今はアンチパターン」となることが多い。
が、Goの場合はGo1.X系の間はほぼ問題ないだろう（並行処理やパッケージ管理は除く）。

言語仕様に破壊的な変更が入るのはGo2からだが、Go2は4年後（Go 1.20）と言われている[*](https://blog.golang.org/toward-go2)ので、（pkgはこれからも更新があるだろうが、）今の仕様を理解しているだけでよい。

このような背景があり、検索に埋もれがちになった過去の情報から、「Goらしく書くため」の情報を探ってみようという運びになった。

Simplicity（単純さ）

- Go言語によるWebアプリケーション開発_付録B
訳者の鵜飼さんが本文中のコードをよりGoらしく書き直している。


# 言語思想

## プログラミング言語Go
- https://amazon.jp/dp/4621300253

## Go at Google: Language Design in the Service of Software Engineering
- https://talks.golang.org/2012/splash.slide#1
- https://talks.golang.org/2012/splash.article

Googleが直面していた開発の課題。
何を解決するために/何を解決したくてGoが作られたのか

## Simplicity is Complicated
- https://talks.golang.org/2015/simplicity-is-complicated.slide#1

## Toward Go 2
- https://blog.golang.org/toward-go2

- Go2へのロードマップ
- Goの仕様がどう策定されるのか
- ステップには意見の公募も含まれているので要不要の意見はGo Teamに送ろう

## The Go Gopher
- https://blog.golang.org/gopher


Gopherの生い立ち。
Go's new brandの中でもGopherを書くときの条件が書いてある

## Practical Go
- https://dave.cheney.net/practical-go

Dave Cheney氏のブログ記事のオススメリンク集
Functional optionsパターンや`T`型メソッドと`*T`型メソッドの使いわけなど
ロギングやエラーハンドリングも含めて開発で一度は悩むポイントを解説してくれている

## Go Proverbs
- https://go-proverbs.github.io/

Rob Pike氏の格言集。Gopher Slackのロード時にも見れたりする。
その格言が出た発表はYouTubeで確認することができる。
言語設計者が何を解決したくてこの仕様にしたのか。
何を意図してこの仕様にしたのか


# Go Wayな設計・実装


## Effective Go
- https://golang.org/doc/effective_go.html

## CodeReviewComments
- https://github.com/golang/go/wiki/CodeReviewComments

# Go言語によるWebアプリケーション開発
- https://amazon.jp/dp/4873117526

## Goに入ってはGoに従え
- https://ukai-go-talks.appspot.com/2014/gocon.slide#1

## Two Go Talks: "Lexical Scanning in Go" and "Cuddle: an App Engine Demo"
- https://blog.golang.org/two-go-talks-lexical-scanning-in-go-and

## Errors are values
- https://blog.golang.org/errors-are-values


## みんなのGo言語[現場で使える実践テクニック]
- https://amazon.jp/dp/4297107279

環境構築
CLIツールの作り方
テストの書き方
まずなにか作ってみたい！という時に最適な一冊
リンク先をクリックすると…


# 終わりに


# 参考文献
- [Go 1 and the Future of Go Programs](https://golang.org/doc/go1compat)
- [Go's New Brand | The Go Blog](https://blog.golang.org/go-brand)
- [「Go言語らしさ」とは何か？Simplicityの哲学を理解し、Go Wayに沿った開発を進めることの良さ](https://employment.en-japan.com/engineerhub/entry/2018/06/19/110000)
- [Go Brand Book](https://storage.googleapis.com/golang-assets/Go-brand-book-v1.9.5.pdf)
- [プログラミング言語Go](https://amzn.to/2IV2Cvo)
- [みんなのGo言語 現場で使える実践テクニック](https://amzn.to/2IXesFH)
- [Go言語によるWebアプリケーション開発 ](https://amzn.to/2WVc4UU)
- [Go at Google](https://talks.golang.org/2012/splash.slide#1)
- [Go at Google: Language Design in the Service of Software Engineering](https://talks.golang.org/2012/splash.article)
- [Simplicity is Complicated](https://talks.golang.org/2015/simplicity-is-complicated.slide#1)
- [Toward Go 2 | The Go Blog](https://blog.golang.org/toward-go2)
- [The Go Gopher | The Go Blog](https://blog.golang.org/gopher)
- [Practical Go](https://dave.cheney.net/practical-go)
- [Go Proverbs](https://go-proverbs.github.io/)
- [Effective Go](https://golang.org/doc/effective_go.html)
- [CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Goに入ってはGoに従え](https://ukai-go-talks.appspot.com/2014/gocon.slide#1)
- [Two Go Talks: "Lexical Scanning in Go" and "Cuddle: an App Engine Demo" | The Go Blog](https://blog.golang.org/two-go-talks-lexical-scanning-in-go-and)
- [Errors are values | The Go Blog](https://blog.golang.org/errors-are-values)




<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=f39eabd7c03a567850294e6c13cb4780"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873117526&linkId=505056985b2190d1573bd05bfe96a507"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B01LMS7B1O&linkId=96b77b09efc6cf8c174af2004e6c772d"></iframe>

