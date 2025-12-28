+++
title= "Goらしさとは何なのか考える"
date= 2020-10-23T09:50:01+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go","esssay"]
tags = ["golang","homebrew","cli"]
keywords = ["Go","homebrew","cli"]
twitterImage = "logos/Go-Logo_Aqua.png"

+++


「Goらしさ」や「Goに入ってはGoに従え」というけれど、「Goらしい」って一体なんだろう？と考えてみる。

<!--more-->

# TL;DR
後半は完全に私見の域を出ない&&身も蓋もない結論なので、最初に参考情報だけまとめておく。

「Goらしさ」が何を目指しているのか、何を目指す考えが「Goらしさ」なのか知りたいならば、まず言語思想・設計思想を知るべきだろう。
言語思想についてまとめられている文書は次の情報だ。

- [Go's New Brand | The Go Blog](https://blog.golang.org/go-brand)
- Go at Google: Language Design in the Service of Software Engineering
    - https://talks.golang.org/2012/splash.slide#1
    - https://talks.golang.org/2012/splash.article

一言でいうと「Goらしさ」とは「Simplicity（簡潔性）」だ。それは次のミッションを実現するための手段だ。

- GoのMission
    - Creating software at scale
    - Running software at scale

Goはあくまでミッションを「スケール」と書いていて、単純な「スピード（開発速度）」ではないところが重要なのではないか。
日本語でまとめられている文書で言うと[@songmu](https://twitter.com/songmu)さんの記事を読むのが良いと思う。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://employment.en-japan.com/engineerhub/entry/2018/06/19/110000" data-iframely-url="//cdn.iframe.ly/gl8gw3N?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

書籍で「Goらしさ」や「Simplicity（簡潔性）」を学ぶならば次の書籍だろう。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=f82c2a7c84c9932f2d9b0e828425e144"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873117526&linkId=28d8f4e493c77b65882433605537618b"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4274064069&linkId=22a6097278849fa652df1ea6f4af2903"></iframe>

言語思想や言語哲学に沿った実装を行えばその言語がもつパフォーマンスや表現力をフルに扱うことができるはずだ。
ここでいうパフォーマンスは実行時の性能だけを指すのではなく、開発中の生産性なども含む。
そのためには当たり前の結論でしかないが、次のような研鑽を積むしかないのかなと思う。

- 言語思想や哲学を学ぶ
- 標準パッケージ、有名なOSSや著名なGopherのコードからGopher仕草を盗み取る
- 一般的なプログラミング設計原則やエンジニアリングのプラクティスを学ぶ

Goはソフトウェアエンジニアリングを行なうための言語である[^1]ため、プラクティスや設計原則に基づいた実装をすることが最適なコードとなる。  
ただ、エラー処理が雑になりがち[^2]な例外機能や、”継承”のような「大半は悪い設計[^3]」になるようなデザインは言語としてサポートされていない。  
**あまり考えずに既知のアプローチを試すと失敗するのでそこは言語仕様や哲学の学習が必要となる**。
上記の書籍以外にもまずは次の文書から読むと良いだろう。

- [Effective Go][ef_go]
- [CodeReviewComments][crc]

[^1]: [Go at Google: Language Design in the Service of Software Engineering 3. Go at Google](https://talks.golang.org/2012/splash.article)
[^2]: 基底Exceptionクラスでtry-catchしたりしてませんか。
[^3]: 継承する前にEffective Javaを読もう。[項目16 「継承よりコンポジションを選ぶ」　[プログラマー現役続行]](https://yshibata.blog.ss-blog.jp/2009-12-05)

その他の基礎情報は次の記事にまとめてある。

- [[発表資料] 今改めて読み直したい Go基礎情報 その1 #golangtokyo](/2019/06/20/golangtokyo25-read-again-awesome-go-article/)


# どうなると「Goらしさ」があるのか？
まず、「Go言語らしくxxxxする」と言うとき、Mustなこと（全員一致なGoらしさ）とWantなこと（人によりそうなGoらしさ）があると思う。
そして、Wantな部分は結構経験によったり、人によって解釈が違うこともあるので難しい。

## MustなGoらしさ
まず必ず守らなければいけないGoらしさがある。

- `gofmt`をかける
- `panic`を使わない
- `error`を握りつぶさない

`gofmt`をかけたりする`panic`を避けることに異論があるGopherはいないだろう。

## WantなGoらしさ
上記のような当たり前のプラクティスとは別に意見が分かれるような「Goらしさ」もあるのかなと思う。

### たとえばAssertion
たとえば私は`testify`（`assert.Equal`などが定義されている3rdパーティのライブラリ）を使わない。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/stretchr/testify" data-iframely-url="//cdn.iframe.ly/xdoita7?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

私は`testing`パッケージ以外のDSLを覚えるのが嫌いだからだ。
そもそもなぜ標準パッケージで`assertXXX`関数が提供されていないかは書籍「プログラミング言語Go」の11.2.5節で述べられている。

…ただ私は`gomock`（そこそこリッチなモックオブジェクトを作れるgoogleのライブラリ）は使っている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/mock" data-iframely-url="//cdn.iframe.ly/RivSfnz?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`gomock`も独自DSLのメソッドチェーンで呼び出し回数などを指定することができる。
「さっきDSL覚えるの嫌って言っていたじゃん」と言われるとその通りである。

- assertユーティリティが無くてもテストは書ける（困ったことがない）
    - 依存を増やしてまでほしいと思わない
    - 組織の中でhelper関数を作って組織としての知識を醸成するのは`Creating software at scale`に則っていると思う
- モックを毎回自前で作るのはさすがにしんどい（困ったことがある）

自分の中では上記のような判断基準があるのだけれど、「いや`NotContains`関数ないと困るでしょ！！」のような意見もあると思う。

### たとえばFunctional Option Pattern
Goではオーバーロードができないので、引数違いの同名関数、同名メソッドが定義できない。
そこでGoではFunctional Option Patternがよく使われる。

- Self-referential functions and the design of options
    - https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html

これはOpen-Closedの原則に則っている実装パターンだ。
変更に対して関数のシグネチャは閉じているが、オプションの追加という拡張に対しては開いている。
すごいシンプルで私はGoらしくて好きなパターンだが、クロージャを使っていたりするので、もしかしたら「どこがシンプル？」という意見もあるかもしれない。

同様にmiddleware/interceptorパターンも初期化のためのオブジェクトを要するようになると人によっては読みにくいかもしれない。
```go
func NewLoggingMiddeware(l *log.Logger) func(http.Handler) http.Handler {
  return func(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
      buf := &bytes.Buffer{}
      rww := NewRwWrapper(w, buf)
      next.ServeHTTP(rww, r)
      l.Printf("%s", buf)
    })
  }
}
```

また、変数の参照可能性を利用した下記のようなクロージャの宣言も読み手によってはシンプルには見えないかもしれない。
```go
// プログログラミング言語Go 5.6節より引用
func squares() func() int {
  var x int
  return func() int {
    x++
    return x * x
  }
}

// f = squares()
// f() // return 1
// f() // return 4
// f() // return 6
// f() // return 16
```

どれも言語仕様でサポートされているのでGopherとして「Goらしく読みやすい」と考えていいと思うのが私の考え方だ。
こういう考えかたの延長で言うと、2021年以降はGoらしさ with Genericsでいろいろなパターンやベストプラクティスを更新しないといけない。

# Goらしい書き方の参考
Goらしい書き方の参考になるのは標準pkgなどの「そこ」にあるコードだが、どのように考えるべきかは書籍や先人たちの知恵に頼るのがよいだろう。
プログラミング言語Goは文法の他にも言語仕様やちょっとしたプラクティス、設計背景も記載されており、「Goらしさ」を学ぶ上でとても大事な書籍だ。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=f82c2a7c84c9932f2d9b0e828425e144"></iframe>

「Go言語によるWebアプリケーション開発」はだいぶ内容が古い。
が、訳者でgooglerの[鵜飼さん](https://twitter.com/fumitoshi_ukai)がGoっぽくない原著のコードをかなり指摘したり付録で修正している。
「なぜGoらしくないか」の解説もあるので読みがいがある。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873117526&linkId=28d8f4e493c77b65882433605537618b"></iframe>

その鵜飼さんやGooglerが書いたスライドも参考になるだろう。

- Go for gophers | GopherCon closing keynote
    - https://talks.golang.org/2014/go4gophers.slide#1
- Goに入ってはGoに従え | Go Conference 2014 autumn
    - https://ukai-go-talks.appspot.com/2014/gocon.slide#1

一般の設計原則の他に、UNIXという考え方もSimplicity（簡潔性）を考える上で重要だと思う。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4274064069&linkId=22a6097278849fa652df1ea6f4af2903"></iframe>

# 反証。Web開発やCLI作りでGoらしさは必ずしも正しいのか
とはいえ、Goらしさに固執することで生産性が落ちるならば本末転倒なわけで難しい。  
Web開発ではDDDやクリーンアーキテクチャなどの考えやgRPC、OpenAPIやWebフレームワークを採用することもあるだろう。  
Go以外の有効なプラクティスに寄せることでGoらしさが守られなくても生産性が向上するならばそれは問題ないのでないだろうか。

**ライブラリなどを作る場合は厳守しておいたほうがよい。Goのライブラリならばライブラリ利用者に「標準pkg」のような使い心地を提供したほうがよく、Goらしさが重要だろう。**
一方で何かしらのアプリケーションを作る際は作る対象・あるいはチーム内で合意できたアプローチをしてもよい。というのが私の今の気持ちだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/google/exposure-notifications-server" data-iframely-url="//cdn.iframe.ly/8V5ghG0?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

たとえば`util`なんてパッケージ名はGoでは良くないとされるが、Googleが作ったwebサーバを見ても、`pkg/timeutils`パッケージなんかがあり、ほほーん？となる。
要はバランスってやつだ。きっと。


# 終わりに
なにか落としどころが微妙な散文になってしまったが、結局「どうしてそうなっているか、あるいはそうするかを考えながらたくさんコードを読んだり書いたりしてバランスを取ろうね」という身も蓋もない結論になってしまった。

# 参考情報
- [[発表資料] 今改めて読み直したい Go基礎情報 その1 #golangtokyo](/2019/06/20/golangtokyo25-read-again-awesome-go-article/)
- [Go's New Brand | The Go Blog](https://blog.golang.org/go-brand)
- Go at Google: Language Design in the Service of Software Engineering
    - https://talks.golang.org/2012/splash.slide#1
    - https://talks.golang.org/2012/splash.article
- [「Go言語らしさ」とは何か？　Simplicityの哲学を理解し、Go Wayに沿った開発を進めることの良さ](https://employment.en-japan.com/engineerhub/entry/2018/06/19/110000)
- [Effective Go][ef_go]
    - https://golang.org/doc/effective_go.html
- [CodeReviewComments][crc]
    - https://github.com/golang/go/wiki/CodeReviewComments

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=f82c2a7c84c9932f2d9b0e828425e144"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873117526&linkId=28d8f4e493c77b65882433605537618b"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4274064069&linkId=22a6097278849fa652df1ea6f4af2903"></iframe>

[ef_go]: https://golang.org/doc/effective_go.html
[crc]: https://github.com/golang/go/wiki/CodeReviewComments

# 余談
IT業界では「お気持ち」な記事をポエムっていうけれど、本当は「エッセイ」なんじゃないかなーと思ったりする。

- [ポエムの意味 - goo国語辞書](https://dictionary.goo.ne.jp/word/%E3%83%9D%E3%82%A8%E3%83%A0/)

> １ 1編の詩。韻文作品。
> ２ 詩的な文章。詩的だが中身のない文章・発言を揶揄 (やゆ) していうこともある。

「中身のない文章・発言」だと自虐的に宣言するからポエムなんだろうか？

- [エッセーの意味 - goo国語辞書](https://dictionary.goo.ne.jp/word/%E3%82%A8%E3%83%83%E3%82%BB%E3%83%BC/)

> １ 自由な形式で意見・感想などを述べた散文。随筆。随想。
> ２ 特定の主題について述べる試論。小論文。論説。


そういう意味だと、中身はあるつもりで書いているので、「自由な形式で意見・感想などを述べた散文。随筆。随想」という意味のエッセイということにしたい。
（といいつつタグはポエムでつくってしまっている。）
