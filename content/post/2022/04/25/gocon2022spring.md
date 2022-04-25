+++
title= "Go Conferrence 2022 Springで登壇した #gocon"
date= 2022-04-25T17:00:55+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go","report"]
tags = ["go","gocon","test"]
keywords = ["Go","GoCon","goconA","test"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

先日行われたGo Conferrence 2022 Springで「testingパッケージを使ったWebアプリケーションテスト」というタイトルで登壇した。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://gocon.jp/2022spring/sessions/a10-s/" data-iframely-url="//iframely.net/rILEO0q?card=small"></a></div></div><script async src="//iframely.net/embed.js" charset="utf-8"></script>

<!--more-->

# Go Conferrence 2022 Spring
https://gocon.jp/2022spring/

<div class="iframely-embed"><div class="iframely-responsive" style="padding-bottom: 52.5%; padding-top: 120px;"><a href="https://gocon.jp/2022spring/" data-iframely-url="//iframely.net/sszkFcs"></a></div></div><script async src="//iframely.net/embed.js" charset="utf-8"></script>

※ The Go gopher was designed by [Renee French](http://reneefrench.blogspot.com/). Illustrations by [tottie](https://twitter.com/tottie_designer).

Go Conferrence（GoCon）は半年に1回ペースで行われる日本国内最大級のGoのイベントだ。  
Paper Callで募集要項（CFP）が公開され、プロポーザルを審査された結果登壇できるかが決まる。私としては[2019年夏ぶり](https://speakerdeck.com/budougumi0617/gocon-fukuoka-2019-summer)に登壇できた。

- https://www.papercall.io/gocon-2022-spring

# 発表資料
当日に使った発表資料はこちら。  
https://speakerdeck.com/budougumi0617/gocon2022spring

<script async class="speakerdeck-embed" data-id="5ce760a80d224a869ecb6b8c73959a94" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

## 資料内のリンク
- P7: Google Testing Blog: Just Say No to More End-to-End Tests
    - https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html
- P7: Using Models to Help Plan Tests in Agile Projects | Agile Testing Quadrants | InformIT
    - https://www.informit.com/articles/article.aspx?p=2253544&ranMID=24808
- P16 Repositoryによる抽象化の理想と現実
    - https://speakerdeck.com/sonatard/ideal-and-reality-of-abstraction-by-repository
- P26: マイクロサービスの開発とテストファースト／テスト駆動開発 【Mercari Gears Lecture Series】
    - https://engineering.mercari.com/blog/entry/gears-microservices/
- P33: Go Fridayこぼれ話：非公開（unexported）な機能を使ったテスト
    - https://engineering.mercari.com/blog/entry/2018-08-08-080000/ 
- P33: Go言語でのテストの並列化 〜t.Parallel()メソッドを理解する〜
    - https://engineering.mercari.com/blog/entry/how_to_use_t_parallel/
- OSS
    - https://github.com/golang/mock
    - https://github.com/budougumi0617/cmpmock
    - https://github.com/DATA-DOG/go-sqlmock
    - https://github.com/google/go-cmp
    - https://github.com/k1LoW/octocov

# 話した内容について

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://t.co/xtgr6VsCuu">https://t.co/xtgr6VsCuu</a><br>タイトルからするとちょっとしたHow-toかと思ったけど、リアルな事情に基づいた解決方法や妥協点の話でとても良かった<a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; gim_kondo (@gim_kondo) <a href="https://twitter.com/gim_kondo/status/1517760122449457152?ref_src=twsrc%5Etfw">April 23, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">そのプロダクトで効果のあるテスト、検証したいことを意識するというのはとてもいいですね<a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a> <a href="https://twitter.com/hashtag/goconA?src=hash&amp;ref_src=twsrc%5Etfw">#goconA</a></p>&mdash; dyuji (@dyuji1) <a href="https://twitter.com/dyuji1/status/1517759953347674113?ref_src=twsrc%5Etfw">April 23, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">なんのためにテストを実装するのか? を明らかにして、<br>どこの責務に対してテストを書いてのか? がきちんと分割されてるの良き👏<a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a> <a href="https://twitter.com/hashtag/goconA?src=hash&amp;ref_src=twsrc%5Etfw">#goconA</a></p>&mdash; im || imaima (@im_miolab) <a href="https://twitter.com/im_miolab/status/1517758936220258304?ref_src=twsrc%5Etfw">April 23, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

タイトルのとおり、テストの話をしたのだが、「なぜこのようなテストを行なっているのか？」を話の主軸においた。  
テストは書いたほうが良いに決まっているが、テストを書くことで何を保証・検証したいのか？によって必要なテストの形式は変わってくる。  
自分のチームはどのような考えてアプリケーションを設計し、そのアプリケーションの品質を保つためにどのようなテストを書いているのかが伝わっていれば幸いである。

## 資料作成はMiroとmarkmapがよかった。
今回の発表資料の作成ではまず書籍から一般的な「Why」の考えを集める作業をした。
関連書籍をばーっと見てMiroで一人ブレストするのが結構よい感じだった。
これをしないとただのHowTo的な発表になっていたと思うのでやってよかった。

![Miroでブレスト](/2022/04/gocon_miro.png)

これのあと[@t_wada](https://twitter.com/t_wada)さんが書いていたmarkmapを使ってスライド構成を思案した。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Ft-wada.hatenablog.jp%2Fentry%2Fautomated-test-and-tdd-sd202203" title="「自動テストとテスト駆動開発、その全体像」を執筆しました（Software Design 2022年3月号） - t-wadaのブログ" class="embed-card embed-blogcard" scrolling="no" frameborder="0" style="display: block; width: 100%; height: 190px; max-width: 500px; margin: 10px 0px;"></iframe>

https://markmap.js.org/

Googleスライド上で作業する段階はもう見た目のことくらいしか気にしなくてよくなったので、だいぶ迷わずスライドづくりができた。  
後述の登壇がすぐ控えているので次のスライドもこの流れで作っていきたい。


# 感想
でかいカンファレンスでの登壇は家庭の事情で久しぶりだったので結構緊張した。
案の定コロナ関係で登壇準備にかけるつもりだった時間がほとんど取れなかったのでなんとか発表できてよかった。  
そのせいで他の皆さんの発表をほとんど見ることができなかったのであとでアーカイブや資料を拝見させていただく。  
また、今年は少し家庭の事情が落ち着いてくるはずなので運営のお手伝いも再開したい。

---

また、現職としてもスポンサーを行なったり、同僚も登壇できたのでよかった。

<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fgocon2022s" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

そして今週はGoConスポンサー企業有志による非公式アフタートークイベントもある。私はNew Relicに関する発表をする予定。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://andpad.connpass.com/event/243953/" data-iframely-url="//iframely.net/utEwNBu"></a></div></div><script async src="//iframely.net/embed.js" charset="utf-8"></script>

良い発表をして連休を迎えたい。

# ちょっとした宣伝
登壇中でも触れたが、GoのWebアプリケーション開発の中で必要となる今回のようなテストのようなテクニックや多用されるプラクティスをまとめた本を執筆しているので発表を良いと思ってくれた方は予約しておいてもらえるとより具体的な内容を伝えられると思う。  
こちらは絶賛執筆中。

<iframe sandbox="allow-popups allow-scripts allow-modals allow-forms allow-same-origin" style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4863543727&linkId=18fce7b7b663b31e4d40430f095458bb"></iframe>

# 関連
- [Go Conferrence 2022 Spring](https://gocon.jp/2022spring/)
- [＼非公式／ Go Conference 2022 Spring スポンサー企業4社 アフタートーク](https://andpad.connpass.com/event/243953/)
- [Go Conference 2022 Spring Onlineにシルバースポンサーで協賛・2名登壇します - BASEプロダクトチームブログ](https://devblog.thebase.in/entry/gocon2022s)
- [詳解Go言語Webアプリケーション開発](https://www.amazon.co.jp/dp/4863543727)

# 参考資料
なお、今回の登壇資料を作成するまえに考えをまとめるために読んだ本は次のとおり。
久しぶりに読む名著から最近の本までいろいろ読む機会にもなったのでよかった。

- [クリーンコード](https://amzn.to/3k7DB3f)
- [達人プログラマー](https://amzn.to/3EPEqHG)
- [レガシーコード改善ガイド](https://amzn.to/38kuxVZ)
- [レガシーソフトウェア改善ガイド](https://amzn.to/3v9OpnI)
- [Effective Software Testing](https://amzn.to/3rNaYfW)
- [プログラミング言語Go](https://amzn.to/3k7ewFH)
