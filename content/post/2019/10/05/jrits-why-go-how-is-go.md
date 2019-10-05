+++
title= "[発表資料] 第138回RITS技術交流会『なぜ私たちはGoを書くのか。今あらためて考えるGo言語の良さと実際』"
date= 2019-10-05T09:28:18+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go", "presentation"]
tags = ["golang"]
keywords = ["RITS技術交流会", "Golang", "Go", "Go言語"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

前職主催の勉強会でGoがどんな言語が発表した。

<!--more-->

# イベント情報
登壇したイベントは以下の通り。  
この技術交流会はエンジニアリング・ITビジネスに関する有識者を招いて幅広く技術の学習を行なう集まりで、私が入社する以前から（、退職後の現在まで）続く歴史ある勉強会だ。  
今回はGoに関して紹介してほしいとのことだったので私が登壇することとなった。

||第138回RITS技術交流会|
|---|---|
|URL|https://rits-techforum.connpass.com/event/146462/|
|会場|リコーITソリューションズ株式会社 本社|
|日時|2019/10/04(金) 16:00 〜 17:30|

# 発表内容について

当日発表した資料は以下になる。

<script async class="speakerdeck-embed" data-id="8e59205b85fe4b6dab382566983f0e33" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

今回はGoを知らない人向けに構成している。
構文だったり、Wikipediaを見てわかるようなことばかりだと登壇して説明する意味がないと思っている。
なので、知っている人じゃないとググって出てこないような、「なぜ生まれたのか？」「XXXはなぜこうなっているのいか？（なぜXXXはGoにはないのか？）」という疑問を解決できる、言語哲学やGoの開発背景を中心にまとめた。

また、参加者はほぼリコーグループ（精密機器メーカー）勤務のみなさんなので、Web系以外、ベンチャー以外でもGoは使われていること、決済などのミッションクリティカルなサービスでもGoが使われていることを強調する内容になっている。

# 感想・反省
在職中は参加する側だったので、登壇できたのは嬉しかった。
（時間があまったせいもありそうだが、）質問もいろいろいただけたので、1石投じられた感はあった。
登壇内容自体とは関係ないが、後輩やお世話になった先輩（上司）にも見ていただけたのもよかった。

1時間(最大1時間半)の持ち時間で登壇したのは初めてだったが、発表時間は45分程度の短時間で終わってしまった。時間調整用のおまけスライドを用意したり、デモコーディングを用意したほうがよかった。
私のスキル的に、発表持ち時間が30分を超えると着地時間を調整するのがなかなか難しいので今後対策しておく必要があると感じた。

リモート参加を含めると100名以上の方に見ていただけたようなので、1人でもGoに興味をもってくれた方がいると嬉しい。

# 参考資料
最後にスライドで参照している各種情報を列挙しておく。

- golang.tokyo
  - https://golangtokyo.connpass.com/
- Go Conference
  - https://gocon.connpass.com/
- Go Conference 2019 Autumn
  - https://gocon.jp/
- Go at Google: Language Design in the Service of Software Engineering
  - https://talks.golang.org/2012/splash.article
- Why did you create a new language? | Frequently Asked Questions (FAQ)
  - https://golang.org/doc/faq#creating_a_new_language
- Go's New Brand | The Go Blog
  - https://blog.golang.org/go-brand
- Simplicity is Complicated
  - https://talks.golang.org/2015/simplicity-is-complicated.slide
- Go 2 Draft Designs
  - https://go.googlesource.com/proposal/+/master/design/go2draft.md
- Tour of Go
  - https://tour.golang.org
- Tour of Go (日本語版)
  - https://go-tour-jp.appspot.com/welcome/1
- プログラミング言語Go
  - https://www.amazon.co.jp/dp/4621300253
- 改訂2版 みんなのGo言語
  - https://www.amazon.co.jp/dp/4297107279
- ゴーファーさん -やさしい世界- | LINE Store
  - https://store.line.me/stickershop/product/8100566/ja

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=02661cb77621c1e3f2b2fcf38bf397fd"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4297107279&linkId=ab2a11b94ee9367e8eb8fd949a7c07c0"></iframe>

