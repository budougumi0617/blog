+++
title= "[発表資料] PHPerKaigi2020 PHPでつくるインタプリタ入門 #phperkaigi"
date= 2020-02-10T07:53:31+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["php","presentation"]
tags = ["php"]
keywords = ["PHPerKaigi2020", "PHPerKaigi", "インタプリタ", "Go", "PHP"]
twitterImage = "2020/02/phperkaigi.png"
+++

<!--more-->


||PHPerKaigi 2020|
|---|---|
|URL|https://phperkaigi.jp/2020/|
|会場|練馬区立区民・産業プラザ Coconeriホール|
|日時|2020/02/09, 10, 11|

# 登壇資料
登壇資料は以下です。後日修正などが入り、当日とは一部異なっている可能性があります。

<script async class="speakerdeck-embed" data-id="aab8bb3c355b4fc4b03cad50aec2b72c" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# 参考文献
今回参考にしたインタプリタに関する本・記事は以下の3冊です。
とくに、Go言語でつくるインタプリタは独自言語Monkeyのインタプリタを作る本なのですが、今回のTDDをつかった実装アプローチはほぼこの本を参考にしています。

- 情報系教科書シリーズ コンパイラ
    - https://amazon.co.jp/dp/4274216209
- Go言語でつくるインタプリタ
    - https://interpreterbook.com/
    - https://www.amazon.co.jp/dp/4873118220
- 低レイヤを知りたい人のためのCコンパイラ作成入門
    - https://www.sigbus.info/compilerbook

また、Go言語は以下のURLの1ページに言語仕様がまとめられており、非常にわかりやすいです。

- The Go Programming Language Specification
    - https://golang.org/ref/spec

Goの標準ライブラリが提供しているパーサーのコードとドキュメントは以下にあります。

- https://golang.org/pkg/go/parser/
- https://github.com/golang/go/tree/master/src/go/parser

# 補足・質疑応答など
当日話し足りなかったこと、質疑応答で答えたことなどを追記する予定です。
口頭でなくても、Twitterで[@budougumi0617](https://twitter.com/budougumi0617)までリプライしていただければ回答します。

---

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4274216209&linkId=44f8c6dfdfc015c695086f17831e3f1e"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873118220&linkId=94f0b3fce633e28259d8949adc7621e5"></iframe>
