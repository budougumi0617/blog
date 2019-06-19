+++
title= "[発表資料] 今改めて読み直したい Go基礎情報 その1 #golangtokyo"
date= 2019-06-19T23:02:01+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go", "presentation"]
tags = ["golang"]
keywords = ["Go", "golang"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


golang.tokyo #25で過去の優良情報を振り返る発表を行なった。
この記事は発表で使った資料と口頭で話したことの要約をまとめておく。

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
発表では各々の記事・書籍を紹介する前に、まず今回の発表の背景・Goの言語哲学について触れた。

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

開発とサービスのスケーラビリティを両立するための解として、Goの言語設計者らはSimplicityを答えとしたしそれをGoで実現しようとしている。
このSimplicityの概念を理解すればより「Goらしいコード」を書けそうだ。

## Go1.Xの言語仕様について
Go1.Xは1.0の頃から言語仕様はほとんど変わっていない。Go1.X系の間機能仕様を変えない方針は以下のドキュメントで言及されている。

- Go 1 and the Future of Go Programs
    - https://golang.org/doc/go1compat

> It is intended that programs written to the Go 1 specification will continue to compile and run correctly, unchanged, over the lifetime of that specification. At some indefinite point, a Go 2 specification may arise, but until that time, Go programs that work today should continue to work even as future "point" releases of Go 1 arise (Go 1.1, Go 1.2, etc.).

Release Noteを確認すれば、ほとんど毎回言語仕様は変更されていないことがわかる。

- Release History
    - https://golang.org/doc/devel/release.html

言語仕様が更新された際でもほぼ微調整レベルの変更で、予約語もずっと25個のままだ。  
（どちらが良いかは別として、）他の言語では新しいプログラミングパラダイムをサポートして、古い情報は「○○仕様が増えたから今はアンチパターン」となることが多い。  
が、Goの場合はGo1.X系の間はほぼ問題ないだろう（`context.Context`が増えた並行処理やパッケージ管理はやや注意が必要）。

言語仕様に破壊的な変更が入るのはGo2からだが、Go2は4年後（Go 1.20）と言われている（[注](https://blog.golang.org/toward-go2)）。  
よって、（標準パッケージはこれからも拡張されるだろうが、）言語仕様に絞れば言語思想を理解し、今の仕様を読み込むだけで今後数年は通用する。

このような背景があり、検索に埋もれがちになった過去の情報から、「Goらしく書くため」の情報を探ってみようという運びになった。


# 言語思想
まず、Goの言語思想を学ぶ上で有用な情報を紹介した。

## Go at Google: Language Design in the Service of Software Engineering
- https://talks.golang.org/2012/splash.slide#1
- https://talks.golang.org/2012/splash.article


このスライドと記事にはGoogleが開発の中で何を問題視していたのか、何をGoで解決しようとしたのかが書かれている。
Goをなぜ作ろうとしたのかを知ることでGoのゴールや言語思想を知ることができる。


## プログラミング言語Go（書籍）
- https://amazon.jp/dp/4621300253

プログラミング言語GoはGoチームで静的解析機能部分を多数開発したAlan Donovan氏とK&RのKであり、プログラミング作法などのBrian Kernighan氏が共著をし、
Goの中心メンバーであるRus Cox氏やRob Pike氏がレビューをし、Effective Javaなどを翻訳した柴田芳樹さんが翻訳したGoを一番体系的に学べる日本語書籍だ（柴田さんはレビューもしている）。  
言語仕様や標準パッケージの使い方を学ぶのに最適なのはもちろんのこと、この書籍は「なぜこのような仕様なのか」という点や「この言語仕様をどのように扱うべきか」という点も言及している。  
以下は書籍で触れられている内容の一例だ。


- なぜインターフェースは小さく作るべきか
- Goのカプセル化へのアプローチ
- なぜ`testing` pkgには`setup`/`teardown`や`assert`がないのか
- なぜ例外を持たないのか


## Simplicity is Complicated
- https://talks.golang.org/2015/simplicity-is-complicated.slide#1

Rob Pike氏が発表した上記のスライドでは、次の3点を知ることができる。

- Go言語の哲学とも言える単純さ
- Goの単純さ・シンプルな言語イメージはどのように作られているのか？
- 裏にある複雑さを隠し，利用する人に単純さを提供する意義

例えばGoにおける並行処理は`goroutine`と`channel`で一通りの実装が可能だ。`pthread`などを使ったことがあるともっと複雑な操作をしたくなるかもしれない。
あえて言語機能の中に複雑さを隠蔽することにより、実際に書かれるコードはシンプルなコードになっている。

より詳しい内容は@tenntennさんが書いた当時のレポートで知ることができる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://gihyo.jp/news/report/01/GoCon2014Autumn/0001" data-iframely-url="//cdn.iframe.ly/og7nGG3"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## Toward Go 2
- https://blog.golang.org/toward-go2

この記事ではGo2の仕様を策定するまでのロードマップとして、Goがどのように仕様を策定するかを知ることが出来る。

- Go2へのロードマップ
- Goの仕様がどう策定されるのか
- ステップには意見の公募も含まれている

ロードマップの中ではプロポーザルの提案や意見の募集も含まれている。

- Go 2 Draft Designs
  - https://go.googlesource.com/proposal/+/master/design/go2draft.md

直近では`try`関数なども提案されていて、TwitterやSlackなどで議論を呼んでいる。
ユーザーからの意見も取り入れられるし、英語が苦手な人もSlackの日本語チャンネル（`#japan` in Gopher Slack）などで議論することもできる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/go/issues/32437" data-iframely-url="//cdn.iframe.ly/qLvVzXB"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## The Go Gopher
- https://blog.golang.org/gopher

言語思想（？）ではないかもしれないが、Gopherくんの生い立ち。  
Go's New Brandを見るとGopherくんは結構仕様（？）が厳格で、「モノをもたせるときは指を描かない」「耳の形は…」などがしっかり明記されている。  

- Go Brand Book
  - https://storage.googleapis.com/golang-assets/Go-brand-book-v1.9.5.pdf

## Practical Go
- https://dave.cheney.net/practical-go

Dave Cheney氏のブログ記事のオススメリンク集。Dave氏は`pkg/errors`を開発したり、世界中で登壇もされている方。  
Functional optionsパターンや`T`型メソッドと`*T`型メソッドの使いわけなどの基本的なGoの書き方を丁寧に解説してくれている。  
開発でGoを書くと一度は悩むロギングやエラーハンドリングの記事もあり、ひとつひとつしっかり読んでおきたい。

## Go Proverbs
- https://go-proverbs.github.io/

Rob Pike氏の格言集。Gopher Slackのロード時にも見れたりする。その格言が出た発表のYouTubeへのリンクもされている。  
ひとつひとつが、「そういうことね！」という一言にまとめられているので、うまく咀嚼できない格言は「つまりどういうことなんだろう？」と調べてみると言語思想への理解が深まりそうだ（`Don't Panic`なんて直球もあるが）。  
ちなみに私は`Make the zero value useful.`や`Clear is better than clever.`などがGoらしくて好きだ。

# Go Wayな設計・実装
言語思想の他には具体的な「Goらしいコードを書く」ための情報も紹介した。

## Effective Go
- https://golang.org/doc/effective_go.html

他の言語でもある「Effective」系の情報。Goの特徴的な言語仕様の効果的な使い方がまとめられている。

## CodeReviewComments
- https://github.com/golang/go/wiki/CodeReviewComments

Effective Goに更に加えてベターコーディングが記載されている。  
誰かにGoを教わるときはEffective Goと合わせて最初にリンクを教えてもらうことが多いのではないだろうか？
こちらは日本語版もある。

- #golang CodeReviewComments 日本語翻訳
  - https://qiita.com/knsh14/items/8b73b31822c109d4c497

## Goに入ってはGoに従え
- https://ukai-go-talks.appspot.com/2014/gocon.slide#1

GoogleでGoのコードをレビューしていた鵜飼さんがまとめたGoらしいコードの書き方。  
`defer`で呼びたい関数の`error`戻り値のハンドリング方法や、`Mutex`を使わずにgoroutine`で解決する方法など、実際にどうGoを駆使すればシンプルで簡潔なコードが書けるのかを解説してくれている。

この発表も@tenntennさんが書いた当時のレポートがあるので、そちらを参考にするとより理解が深まる。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://gihyo.jp/news/report/01/GoCon2014Autumn/0002" data-iframely-url="//cdn.iframe.ly/fLmSd1T"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# Go言語によるWebアプリケーション開発
- https://amazon.jp/dp/4873117526

本篇はAPIやMongoDBを使ったアプリ開発で、そちらの内容も非常に勉強になるのだが、日本語版は監訳者の鵜飼さんの「Goらしいコードの書き方」が付録についている。  
付録では本文中のコードをよりGoらしく書き直している。
Twitter APIやDB操作といった外部操作を含んだボリュームがある関数をリファクタリングする例が載っており、より実践的なリファクタリングアプローチを知ることができる。
`Go1.17`以前の書籍だが、Contextを使ったリファクタリング例もある。


## みんなのGo言語[現場で使える実践テクニック]
- https://amazon.jp/dp/4297107279

まずGoを書いてみるか！というときに最適な書籍。日本でバリバリGoを書いているみなさんが各章担当している。  
以下のような「まずなにか作ってみたい！」という時に必要な情報が一冊にまとまっている。

- エディタなどの環境構築
- CLIツールの作り方
- テストの書き方

そのため、「ひとまず一冊本買って始めてみるか」というときに最適だし、中級者以上でも「これは知らなかった」となることが書いてある。  
ちなみに8月に改訂版が出版されるらしく、Go Modulesの対応なども含まれるとのこと。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://mattn.kaoriya.net/software/lang/go/20190618181623.htm" data-iframely-url="//cdn.iframe.ly/8ln3nfz"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 終わりに
私はGoを始めて書いたのが2016年ごろだったので、今回他の運営メンバーにいろいろな記事を教えてもらいよりGoの言語思想の背景を知ることができた。  
なかなか言語設計や言語思想まで調べようと思うことはないかもしれない。が、背景を知ることで設計アプローチや種々の実装の判断もよりスマートなコードが書けるようになると思う。  
今回少し時間がなく、ひとつひとつの情報を読み込んだり、関連リンクまではきちんと追えていないので、あとで改めて読み直したい。

なお、より実装や個別の機能についての優良情報は@izumin5210さんがまとめてくれている。

<script async class="speakerdeck-embed" data-id="b36992074ba343ada06059e73a02a583" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# 参考文献
- [今改めて読み直したい Go基礎情報 その1](https://speakerdeck.com/budougumi0617/read-again-awesome-go-article)
- [今あらためて読み直したい Go 基礎知識 その2](https://speakerdeck.com/izumin5210/golang-dot-tokyo-number-25)
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
- [Go言語の父と呼ばれるRob Pike氏による基調講演～Go Conference 2014 Autumn基調講演1人目](https://gihyo.jp/news/report/01/GoCon2014Autumn/0001)
- [Toward Go 2 | The Go Blog](https://blog.golang.org/toward-go2)
- [The Go Gopher | The Go Blog](https://blog.golang.org/gopher)
- [Practical Go](https://dave.cheney.net/practical-go)
- [Go Proverbs](https://go-proverbs.github.io/)
- [Effective Go](https://golang.org/doc/effective_go.html)
- [CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Goに入ってはGoに従え](https://ukai-go-talks.appspot.com/2014/gocon.slide#1)
- [鵜飼文敏氏「Goに入ってはGoに従え」可読性のあるコードにするために～Go Conference 2014 Autumn基調講演2人目](https://gihyo.jp/news/report/01/GoCon2014Autumn/0002)
- [Two Go Talks: "Lexical Scanning in Go" and "Cuddle: an App Engine Demo" | The Go Blog](https://blog.golang.org/two-go-talks-lexical-scanning-in-go-and)
- [Errors are values | The Go Blog](https://blog.golang.org/errors-are-values)
- [Big Sky :: 改訂2版 みんなのGo言語](https://mattn.kaoriya.net/software/lang/go/20190618181623.htm)



<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=f39eabd7c03a567850294e6c13cb4780"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873117526&linkId=505056985b2190d1573bd05bfe96a507"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B01LMS7B1O&linkId=96b77b09efc6cf8c174af2004e6c772d"></iframe>

