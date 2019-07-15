+++
title= "[発表資料] GoCon Fukuokaでgoogle/wireを使った Goらしいアーキテクチャへの取り組みを発表した #gocon"
date= 2019-07-15T17:33:07+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go", "presentation", "wire"]
tags = ["golang", "gocon", "wire", "design"]
keywords = ["Go", "Golang", "アーキテクチャ", "google/wire", "wire"]
twitterImage = "2019/07/15_ogp.png"
+++

GoCon '19 Summer in Fukuokaで登壇してきたので感想と参考資料などをまとめた。

<!--more-->

|イベント名|GoConference '19 Summer in Fukuoka |
|---|---|
|URL|https://fukuoka.gocon.jp/ja/|
|Connpass|https://fukuokago.connpass.com/event/130797/|
|会場|福岡市中央区天神 Fukuoka Growth Next|
|日時|2019/07/13（土）10:00-19:00|
|ハッシュタグ|[#gocon](https://twitter.com/hashtag/gocon)|
|Toggeter|https://togetter.com/li/1375799|

当日のスライド資料は以下となる。

<script async class="speakerdeck-embed" data-id="16fb9854de5f42d19d1786ba7b0d1dbe" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# 資料を作る上で参考にした情報
## google/wireについて
google/wireについては公式リポジトリのドキュメントを見るのが一番良い。
ベータリリースのため、(私の記事も含めて)ブログ記事などの情報は最新情報ではない可能性がある。
特に6月にリリースされたv0.3.0では破壊的変更が入り、`wire.Struct`などの新しい概念も入っている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/google/wire/blob/master/docs/guide.md" data-iframely-url="//cdn.iframe.ly/X4QqTGW"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

日本語の記事だと以下の記事がわかりやすいと思う。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://blog.ishkawa.org/2018/12/10/1544371624/" data-iframely-url="//cdn.iframe.ly/Krknhj9"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevelopers.freee.co.jp%2Fentry%2Fservice-infra-and-wire" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no" allow="encrypted-media"></iframe>


また、このブログでも何回か取り上げている。

- [wireタグの記事一覧](/tags/wire/)

## Goらしい設計について
Goらしいとは「Simplicity」だと考えている。言語思想については@songmuさんがまとめてくださっている。
（[@songmu](https://twitter.com/songmu)さんにはGoCon当日の懇親会などでもたくさんご助言いただいた。感謝。）

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Femployment.en-japan.com%2Fengineerhub%2Fentry%2F2018%2F06%2F19%2F110000" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no" allow="encrypted-media"></iframe>

また、英語でも問題ない方はGoのブランドブックや、コアメンバーであるRob Pike氏の話をみればよいだろう。

- Go Brand Book v1.9.5
  - https://storage.googleapis.com/golang-assets/Go-brand-book-v1.9.5.pdf
- Simplicity is Complicated
  - https://talks.golang.org/2015/simplicity-is-complicated.slide#1


## 実際のアーキテクチャ設計について
ここに関してはアーキテクチャの本関連をもろもろ読み、社内の強いエンジニアの人のレイヤー構成を参考にしているとしか言えないところだ…。
GitHubにサンプルコードを後日公開し、背景や意図を書くことにする。

強く影響を受けている書籍などは以下の通り。


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4048930656&linkId=46265df25801829599dc7dc7c7354de7"></iframe>

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4274064069&linkId=36ff223e5baa6f6bb2b90b57703a8b88"></iframe>

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B00LGJTXT8&linkId=6a073d547da667a6b27b8456a23bc7d3"></iframe>

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://booth.pm/ja/items/1045782" data-iframely-url="//cdn.iframe.ly/8L1q4X1"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2F2018%2F11%2F26%2F102401" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no" allow="encrypted-media"></iframe>

# 感想
## 自分の発表について
今回は20分のつもりで準備をしていたが、諸々の事情で15分ほどしか時間がなかったため最後まで発表することができなかった。

- 前の発表の発表まで押していた&&機材トラブルで開始が遅れていた
- 次の発表者が福岡市長だったので予定終了時間は動かせなかった

話す直前に事情はわかっていたので話す内容を調整できればよかった。  
が、google/wireの認知度が予想以上に低かったりで予定以上に前半で時間を使ってしまった。  
発表練習が足りなかったのは事実なので、もっと練習をしておけば対処できた。  
サンプルコードも動く形にしてGitHubには公開できていないので、供養がてらブログ記事として後日公開したい。

## GoCon Fukuokaについて
会場となったFukuoka Growth Nextは小学校の旧校舎を改装した施設で、懐かしい感じとリノベーションされた雰囲気が混在した面白い空間だった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://growth-next.com/ja/" data-iframely-url="//cdn.iframe.ly/N13S9pq?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

運営のみなさん、スポンサー企業の方々もとっても気合が入っており、トートバックや登壇者向けTシャツをいただけた。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">GoCon始まるよ！！ <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a> <a href="https://twitter.com/hashtag/fukuokago?src=hash&amp;ref_src=twsrc%5Etfw">#fukuokago</a> <a href="https://t.co/K0djZYI4al">pic.twitter.com/K0djZYI4al</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1149842855315197952?ref_src=twsrc%5Etfw">July 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

前述したとおり福岡市長も登壇し以下のような内容をプレゼンしてくださった。

- どんな文化を持った街なのか
- 如何に企業活動に向いているか
- エンジニア（IT企業）と一緒にどう発展していきたいか

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">福岡はアジアの中心！博多2.5km圏内に空港、新幹線の駅、港があるぞ！ <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1149872643614494720?ref_src=twsrc%5Etfw">July 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

当日の市長の発表内容は既にYouTubeに公開されている。さすがだ。

- 福岡市長高島宗一郎 GO conference'19 summer in fukuokaに出席しました
  - https://youtu.be/NDbg3QffW4k

午後からはチームラボさん提供の生ビールもデプロイされていた。
懇親会では地元の学生さんらとも話すことができ、福岡の勢いは今後も続くんだろうなと思った。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">さっぽろ＠福岡 <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a> <a href="https://t.co/eBRIS0Eji2">pic.twitter.com/eBRIS0Eji2</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1149903218714877952?ref_src=twsrc%5Etfw">July 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

登壇者は懇親会と別にヌーラボさん提供で打ち上げで粋いかなどを食べさせていただき、朝から晩まで福岡を満喫させていただいた。
また、懇親会のあとは[@hgsgtk](https://twitter.com/hgsgtk)さん、[@po3rin](https://twitter.com/po3rin)さんと念願の屋台も満喫させていただいた。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">GoConお疲れ様でした！！！ <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a> <a href="https://twitter.com/hashtag/nulab?src=hash&amp;ref_src=twsrc%5Etfw">#nulab</a> <a href="https://twitter.com/hashtag/speakerdinner?src=hash&amp;ref_src=twsrc%5Etfw">#speakerdinner</a> <a href="https://t.co/4PexXN68mp">pic.twitter.com/4PexXN68mp</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1150002487224459265?ref_src=twsrc%5Etfw">July 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

そして、今回は終わりにGoCon Tokyo Autumnのスケジュールも発表された。

- Go Conference 2019 Autumn
  - 2019年10月28日(月) 墨田区みどりコミュニティセンター
  - https://gocon.jp/

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Go Conference 2019 Autumn on Oct 28th JST. CFP will open tomorrow. We welcome your proposals!! <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a> <a href="https://t.co/XLGbLoh1Kv">https://t.co/XLGbLoh1Kv</a></p>&mdash; Yoshi Yamaguchi 🇯🇵 (@ymotongpoo) <a href="https://twitter.com/ymotongpoo/status/1149965985035636737?ref_src=twsrc%5Etfw">July 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

秋もプロポーザルを出してぜひ登壇したいと思う（そして運営としても参加するので福岡に負けないようないい会にしないと）。

# 終わりに
初めて関東以外のイベントに登壇したが、最初から最後まで楽しく過ごせたし、多くのみなさんと交流できて大変学びになり、やる気ももらえた。  
私もいくつかコミュニティの運営に参加しているので今回のGoConFukuokaのように参加者・登壇者が大満足するような会をしていきたい。  

また、GoConの前日はメルカリさんの福岡オフィスからGo Open Fridayにも参加させていただき、参加者のみなさんと和気あいあいGoの悩みだとか雑談（？）をして盛り上がった。

- #5 Open Go Friday in Fukuoka
  - https://mercari.connpass.com/event/136989/

Go三昧で福岡を過ごしたが、振り返ってみるとラーメンもモツ鍋も食べ忘れてしまったのでぜひまた福岡に行ってリベンジしたい。

# 関連資料
- [Go Conference ‘19 Summer in Fukuoka](https://fukuoka.gocon.jp/ja/)
- [#5 Open Go Friday in Fukuoka](https://mercari.connpass.com/event/136989/)
- [google/wireを使った Goらしいアーキテクチャへの取り組み](https://speakerdeck.com/budougumi0617/gocon-fukuoka-2019-summer)
- [github.com/google/wire](https://github.com/google/wire)
- [Wire: Goのコード生成によるDIツール](https://blog.ishkawa.org/2018/12/10/1544371624/)
- [freeeのマイクロサービス基盤とWire導入](https://developers.freee.co.jp/entry/service-infra-and-wire)
- [「Go言語らしさ」とは何か？　Simplicityの哲学を理解し、Go Wayに沿った開発を進めることの良さ](https://employment.en-japan.com/engineerhub/entry/2018/06/19/110000)
- [Go Brand Book v1.9.5](https://storage.googleapis.com/golang-assets/Go-brand-book-v1.9.5.pdf)
- [Simplicity is Complicated](https://talks.golang.org/2015/simplicity-is-complicated.slide#1)
- [pospomeのサーバサイドアーキテクチャ（PDF版）](https://booth.pm/ja/items/1045782)
- [Clean Architecture 達人に学ぶソフトウェアの構造と設計](http://amazon.jp/dp/4048930656)
- [APIデザインの極意 Java/NetBeansアーキテクト探究ノート](http://amazon.jp/dp/B00LGJTXT8)
- [UNIXという考え方―その設計思想と哲学](http://amazon.jp/dp/4274064069)
- [Goを運用アプリケーションに導入する際のレイヤ構造模索の旅路 | Go Conference 2018 Autumn 発表レポート](https://devblog.thebase.in/entry/2018/11/26/102401)
- [福岡市長高島宗一郎 GO conference'19 summer in fukuokaに出席しました](https://youtu.be/NDbg3QffW4k)
- [Go Conference 2019 Autumn](https://gocon.jp/)
