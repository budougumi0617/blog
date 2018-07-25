+++
title = "[発表資料]初級者向けGoの落とし穴と解説 #gounco"
date = 2018-07-26T08:00:00+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["go", "presentation"]
tags = ["golang"]
+++

Go(Un)Conference（Goあんこ）LT大会 3kgの発表資料と資料中の参考リンク、補足をまとめた。


<!--more-->

|||
|---|---|
|URL|https://gounconference.connpass.com/event/92794/|
|会場|株式会社アイスタイル 東京都港区赤坂1-12-32(アーク森ビル34F)|
|日時|2018/07/26(木) 19:30 〜 22:00|
|ハッシュタグ| [#gounco](https://twitter.com/hashtag/gounco)|


---

<script async class="speakerdeck-embed" data-id="3286d81766704757ac4e3ec18048752b" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# Gopherについて
Goの新しいブランドロゴについては以下のブログで公開された。

- 26 April 2018 Go's New Brand
  - https://blog.golang.org/go-brand

記事中にブランドイメージついてまとめたPDFがあり、その中でGopherについても言及されている。

- https://golang.org/s/brandbook

なお、今回のスライドも上記記事で公開されたスライドマスターを利用して作成した。

- https://golang.org/s/presentation-theme

# スライド中のplaygroundのリンク

スライド中のコードはplayground上で実行できるので、実行してみたり、どう修正すれば問題が解消するのか確認にすることもできる。

|スライドタイトル| playgroundのURL|
|---|---|
|例:曖昧な理解だとハマること1|https://play.golang.org/p/A8fouhhtLQZ|
|例:曖昧な理解だとハマること2|https://play.golang.org/p/PUoHxeT-RJu|
|Loop with defer|https://play.golang.org/p/q_V9E1H5Jhy|
|Map|https://play.golang.org/p/rKforjFi2CW|
|Interface|https://play.golang.org/p/QzK9wJQXwcS|
|Captured value|https://play.golang.org/p/24O7H2cQZLf|
|Local variable 1|https://play.golang.org/p/ljulBLJZs6k|
|Local variable 2|https://play.golang.org/p/6NH44WjXOz-|
|Size of empty struct is zero|https://play.golang.org/p/nZqAFEZs48J|
|Visibility private field|https://play.golang.org/p/u8-5-kWyzGI|

# 参考書籍
日本語のGoの書籍ならばプログラミング言語Goの他にも以下がよいと思っている。

- プログラミング言語Go
  - http://amazon.jp/dp/4621300253
- みんなのGo言語【現場で使える実践テクニック】
  - http://amazon.jp/dp/477418392X
- Goならわかるシステムプログラミング
  - http://amazon.jp/dp/4908686033

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=60c659a7abb2c7dda3a8bda5b132808f"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B01LMS7B1O&linkId=ba2078cb70074d6995f4778e3da04293"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4908686033&linkId=ca1623768c07f1a972fcf9c6e0acd040"></iframe>

# 関連
- [当ブログのGo記事一覧](/categories/go/)
- [[書評]Goならわかるシステムプログラミングを読んだ](/2018/02/26/review-go-system-programming)

