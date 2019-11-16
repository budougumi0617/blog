+++
title= "Go Release 10 Year Anniversary Party in Tokyo参加メモ"
date= 2019-11-16T11:18:38+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go","report"]
tags = ["golang"]
keywords = ["Go", "golang", "Go言語", "#GoTurns10"]
twitterImage = "2019/11/16_goturns10.jpg"
+++



メルカリさんで行われたGoリリース10周年パーティに参加したので、遅くなったが参加メモ。  
Goは今年でOSS10周年！
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Hey! Ho! Let&#39;s Go! Open Sourcing the Go programming language http://bit.ly/1NxkxC <a href="https://twitter.com/hashtag/golang?src=hash&amp;ref_src=twsrc%5Etfw">#golang</a> <a href="https://twitter.com/hashtag/developer?src=hash&amp;ref_src=twsrc%5Etfw">#developer</a> <a href="https://twitter.com/hashtag/google?src=hash&amp;ref_src=twsrc%5Etfw">#google</a></p>&mdash; Google Go Team (@golangnuts) <a href="https://twitter.com/golangnuts/status/5606771876?ref_src=twsrc%5Etfw">November 11, 2009</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<!--more-->

||Go Release 10 Year Anniversary Party in Tokyo|
|---|---|
|URL|https://gocon.connpass.com/event/153665/|
|会場|株式会社メルカリ 東京都港区六本木6-10-1 六本木ヒルズ森タワー21F|
|日時|2019/11/11(月) 19:30 〜 22:00|
|ハッシュタグ|[#GoTurns10](https://twitter.com/hashtag/GoTurns10)|
| Toggetter| https://togetter.com/li/1429242|


# TL;DR
Goは10年前の2009年11月10日に世間へ公開された。  
今回はGoが公開されて10周年ということで、（Go以前のプログラミング言語の歴史も含めた）Goの歴史や未来の話を聞くことができた。  
お祝いということで、みなさん普段オープンな場所では言えないような貴重な話をしてくださった。  
上記のとおりなので、書いていいことが少なかったりするのだが、とても良い回だったのでメモしておく。

# History of Go by ymotongpoo
[@ymotongpoo](https://twitter.com/ymotongpoo)さんの発表。

Go言語は2009年10月30日にRob PikeがGoogleの社内プレゼンで初めて登場した。  

- The Go Programming Language | PDF
    - https://talks.golang.org/2009/go_talk-20091030.pdf
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://opensource.googleblog.com/2009/11/hey-ho-lets-go.html" data-iframely-url="//cdn.iframe.ly/kOvtVNU"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


Goが生まれる前のプログラミング言語の歴史、Goの歴史、Goの未来の話。

## Foundations 1960-2007
- 1950年代 ハードウェアベンダー独自のプログラミング言語が主体。各ハードウェア専用の言語しかなかった。
- 1958年`ALGOL`という統一言語が発表された。
  - https://ja.wikipedia.org/wiki/ALGOL
- 以降、様々なプログラミング言語が開発され、多くの系統図がある。

### Concurrencyの進化 1964-2007
- 多くのプログラミング言語が開発され、並行性のアプローチも数多く作られた。
  - `Mutex`、共有メモリベース、メッセージパッシング…
- 78年にはTony HoareがGoの並行処理モデルの原型であるCSPを発表した。
  - https://ja.wikipedia.org/wiki/Communicating_Sequential_Processes

## Refraction 2009-2019
- GoogleのRob PikeはC++のコンパイル時間（45分間）を待っている時間に考えた。
- 今度発表されるC++11を使うようになったら何ができるようになるのか。
  - そんなに機能が増えても本当に必要なのか？今でもコンパイルに45分かかるのに？
- たまたま自席の後ろにいたRobert Griesemerと話し、隣の部屋にいたKen Thompsonも加えて議論がはじまった。
  - `Newsqueak`を開発したRob Pike, C言語のKen Thompson, Oberonを開発し、Pascal作った教授のもとで勉強していたRobertがGoを考えた。
      - https://en.wikipedia.org/wiki/Newsqueak
      - [https://en.wikipedia.org/wiki/Oberon_(programming_language)](https://en.wikipedia.org/wiki/Oberon_(programming_language))
- そこにIan Lance Taylor、Russ Coxが加わりコンパイラも作成された。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">そんな偶然集まるかなww 流石にGoogleというところ <a href="https://twitter.com/hashtag/GoTurns10?src=hash&amp;ref_src=twsrc%5Etfw">#GoTurns10</a></p>&mdash; tenntennʕ ◔ϖ◔ʔ ==Go (@tenntenn) <a href="https://twitter.com/tenntenn/status/1193843072020279296?ref_src=twsrc%5Etfw">November 11, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- Goはいろいろな言語から必要な言語仕様だけを抽出してできた。
  - 想像と収斂を大事にしていて、他の言語のような発展とか複雑化を避けている。
- No is Temporary, Yes is Forever
  - 間違ったものは入れない。

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Rule #1 of open-source: no is temporary, yes is forever.</p>&mdash; Solomon Hykes (@solomonstre) <a href="https://twitter.com/solomonstre/status/715277134978113536?ref_src=twsrc%5Etfw">March 30, 2016</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- 2009年からGenericsの話はあったけどずっと入れなかった。IanはGenericsのドラフトを7回書いて捨てている。
  - Why Generics? | Go Blog
      - https://blog.golang.org/why-generics

### Goの発展と周辺環境
- Goといえばgofmtがキラーツール。みんなが同じフォーマットで書く。
  - 最初から存在していた
  - Goのソースコードをコンパイルすると`go`コマンドと`gofmt`コマンドがビルドされる。
- Go1は2012年にリリース。が、その前からHerokuやApp EngineはGoをサポートしていた。
- 2013年 PackerやCoreOS、DockerがGoで書かれ始めた。
- 2014年 Kuberenetesも公開。その後もTerraformなどCNCFのツールの大多数がGoで書かれている。
  - CNCF Cloud Native Interactive Landscape
      - https://landscape.cncf.io/
- 2014年頃Goは1.5に。ついにGOのコンパイラもGoで書かれるようになり、Goが読めればruntimeも読めるようになった。
  - CoC, WWG, GoBrigeも
      - https://golang.org/conduct
      - https://www.womenwhogo.org/
      - https://forum.golangbridge.org/
- 2016年 Go1.6になり、`context`パッケージが標準パッケージに採用された。
- 2017年にはGCが速くなり、2018年にはvgoも発表された。
- 2019年はコントリビュータが2100人にもなっている。（前年度比30%UP）
- Conference開催数も2018年の2倍になっている。

## Future 2019
- Goをどう勉強すればいいのか？Tour of Go以外の情報もどんどん提供して開発者を支援していきたい
    - 言葉の通り、2019/11/14にポータルサイトが公開された。その中に学習のための情報もまとめられている
        - https://go.dev/
        - 学習コンテンツは https://learn.go.dev/
- Go1.13ではGo Proxyが導入されたり、Go1.14以降もモジュール対応がいろいろ入っていくだろう。
- Go2はどうしても今ある言語仕様を破壊しても入れたい変更が入ったとき。明確にGo2の仕様やリリース時期があるわけではない。
- TinyGoやGioなど今まで話題が少なかった分野へのアプローチも始まっている。
  - https://tinygo.org/
  - https://gioui.org/
- エッジコンピューティングでも使えるかな…？

## 感想
Goが生まれたときの背景は以前からGo at Googleなどを読んでなんとなく知っていた。

- Go at Google
    - https://talks.golang.org/2012/splash.article
- [[発表資料] 今改めて読み直したい Go基礎情報 その1 #golangtokyo](/2019/06/20/golangtokyo25-read-again-awesome-go-article/)

しかし、Rob Pike-sanのコンパイル待ちから始まっていたのは議論がはじめて聞いた。  
なんともめぐり合わせだなと思った。隣の部屋にKen Thompsonがいて、しゅっとコンパイラを作ってしまう Ian TaylorがいるっていうGoogleがすごすぎる。  
`No is Temporary, Yes is Forever`の精神はなかなか強い信念と、自分たちの判断に自信がなくてはできないことだと思う。  
こういったとてつもない偶然と、強い気持ちで支えられて発展しているGoはとても素敵だなと思ったし、もっと好きになった。



# AppEngine Standard for Go の移り変わり sinmetalの思い出から by sinmetal
[@sinmetal](https://twitter.com/sinmetal)さんのAppEngine（以下GAE）にまつわるGoの思い出。

- 最初はApp Engine SDKにGAE専用のGoが同梱されていた。
- 最初のApp EngineはGOPATH以下のソースを全部アップロードするヤバいやつだった。
  - みんなプロダクト別の`GOPATH`を設定したりしていた。
- `context.Context`が導入される前（Go1.7より前）からGAE/Goには`appengine.Context`があった
- Goは依存関係管理ツールが公式ではない時代があり、壊れていても`master`をみてしまうのが辛かった

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">masterは壊れないっていうか、依存先が変更されたら依存してるライブラリは追従しなければいけない、っていう思想があったせいですね <a href="https://twitter.com/hashtag/GoTurns10?src=hash&amp;ref_src=twsrc%5Etfw">#GoTurns10</a></p>&mdash; Yoshi Yamaguchi 🇯🇵 (@ymotongpoo) <a href="https://twitter.com/ymotongpoo/status/1193849252050419712?ref_src=twsrc%5Etfw">November 11, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- GAE用の依存関係管理ツールやテストツールがあったり、独自文化もあった。
  - https://github.com/PalmStoneGames/gb-gae
  - https://cloud.google.com/appengine/docs/standard/go/tools/localunittesting/reference?hl=ja
- `urlfetch`パッケージを使わないと外部APIにアクセスできず、内部的に`http`パッケージを使っているライブラリは（mattnさんのWindows対応並に）GAE/Go対応するPRをガシガシ投げていくしかなかった
- 良くも悪くも今のGAE/GoはGAE固有機能はほぼ消滅して、2nd genでは単純なGoのhttp serverとしてのアプリケーションが作れるようになった


## 感想
GAEは始めよう始めようと思い全然さわれていなかったが、2nd genの話などうっすらとは聞いていた。  
「`appengine`パッケージを使わないといけない独自仕様がちょっとあるよ」というのは知っていたが、昔の苦労時代は全然知らなかった。  
GAEは無料でも始められるので、2019年のうちに私もGAE初デプロイしておきたい。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://cloud.google.com/free/?hl=ja" data-iframely-url="//cdn.iframe.ly/LcdgxtX"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


# Go beyond the frame of eureka by kaneshin
ネットにほとんどGoの情報がないころからEurekaでGoを使っていた[@kaneshin](https://twitter.com/kaneshin0120)さんの発表。

- PairsとGoとモノレポの話。
- GDB辛い問題。
- （どこまで書いていいのかわからないclosedの話だったので詳細は割愛）

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">どこまでtweetしていいのかわからないｗ <a href="https://twitter.com/hashtag/GoTurns10?src=hash&amp;ref_src=twsrc%5Etfw">#GoTurns10</a></p>&mdash; こんぼい (@Konboi) <a href="https://twitter.com/Konboi/status/1193857235392159744?ref_src=twsrc%5Etfw">November 11, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">電子書籍で買った人は申し訳ないけど、無料で『Goの思想とは』を考えた書籍が落とせるのでどんどんシェアしちゃってくださいー！買ってしまった人は来月くらいに内容をアップデートする予定なのでお待ち下さいー<br> <a href="https://twitter.com/hashtag/go?src=hash&amp;ref_src=twsrc%5Etfw">#go</a> <a href="https://twitter.com/hashtag/GoTurns10?src=hash&amp;ref_src=twsrc%5Etfw">#GoTurns10</a> <a href="https://t.co/UyZ7bPV3ab">https://t.co/UyZ7bPV3ab</a></p>&mdash; kaneshin / eureka (@kaneshin0120) <a href="https://twitter.com/kaneshin0120/status/1193888344624660480?ref_src=twsrc%5Etfw">November 11, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 感想
モノレポの話は来月のアドベントカレンダーで執筆する予定とのこと。

- eureka Advent Calendar 2019
  - https://qiita.com/advent-calendar/2019/eureka

私は2016年くらいのGoから知っているが、たしかにその頃でさえあまり情報がなくて業務で使うにはかなりのハードル（地雷は自分で踏んで自分で解決する）があったのかなと思う。
かねしんさんが10周年で公開してくださった電子書籍は以前に購入して読んでいるが本当にオススメ！

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">kaneshinさんの本読んでから、&quot;UNIXという考え方&quot;を読んで、なんでGoの考え方（シンプルさ）が好きなのかよくわかった。<a href="https://t.co/7cRw9fobTA">https://t.co/7cRw9fobTA</a><a href="https://t.co/xUd97Z0BW0">https://t.co/xUd97Z0BW0</a><a href="https://twitter.com/hashtag/GoTurns10?src=hash&amp;ref_src=twsrc%5Etfw">#GoTurns10</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1193858749888249857?ref_src=twsrc%5Etfw">November 11, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# YourCastとHAROiDでのGo利用の遍歴とこれからの展望 by わんこ。
エンタメ業界でGoを使っている[@わんこ。](https://twitter.com/voidofglans)さんの発表。


- 初めてGoで作ったのはソシャゲの詫び石高速付与ツール。
- 現在はYourCastでTV事業をしている。
  - https://techblog.yourcast.co.jp/tag/golang/
  - テレビの双方向コンテンツetc...
- TVで利用していると、放送開始前は0rps（秒間リクエスト）がいきなり数万rpsになる。
- 当然、放送中はSLA100%が絶対。
- 全国5300万のTVがクライアントになる。
- そのような環境でもGoは安心して使えている。
- トリッキーな実装しなくても十分なパフォーマンスがでる。
- 標準パッケージが最強。ベンチ・テストがあるのがいい。
- TRiSH
  - 高負荷専用ETLサービス
  - `json.RawMessage`での遅延評価
  - goroutineとchannelでファイル監視
- 千手観音
  - 10万クライント 20万rpsのシミュレートを$2くらいでできる高負荷検証ツール
  - channelにresult summary
  - OSSにしたい
- GoはCommunityがあるから発展する。Communityと一緒に発展していける。わんこ。さん自身もコミュニティを通して今の会社に務めている


## 感想
SLA100%のような超高可用性が求められる現場での活用事例を聞くことが出来てよかった。  
テストツールなどでも利用されているとのことで、私もgoroutineを湯水のように使うツールを書いてみたいなと思った。  
私自身も11月からコミュニティを通じて転職しているので、Goのコミュニティ活動の繋がり、広がりは非常に感じている。


# Goコミュニティの振り返りとこれから by tenntenn
[@tenntenn](https://twitter.com/tenntenn)さんの国内コミュニティについての発表。

- 初めてGoを触ったのは学生の頃。まだ、golang.orgじゃなかった。
  - http://go-lang.cat-v.org/
- 日本初のGoの勉強会は？
  - 社内勉強会ならば柴田さんが2010年にやったRICOHの勉強会が初めて？
      - https://mercan.mercari.com/articles/15164/
  - 2012年からいろいろ増えてきた。
- 沖縄から北海道まで全国にGoコミュニティができている！
- 各地と中継して勉強会も開催するようになってきた
- いろんな人が参加できる勉強会を
  - Women Who Go Tokyo
  - 初学者向けのハンズオンやコードラボ
  - オンラインで参加できる勉強会
- Go Conferenceの平日開催・有料化について
  - 無料で行なうのは限界が近づいてきた
  - さらに良くしてくために法人化、運営の増強、開催サイクルの見直しを考えている


## 感想
私もgolang.tokyoやGo Conferenceに運営として参加している。  
Go Conferenceは様々な企業のみなさまにもスポンサーしていただけており、Goの広まりはすごいなと思う。  
発表中にもあったが、百人規模のイベントをするのはなかなか大変なので運営に協力していただける方を募集中です。  
（会場調整、各スポンサー様とのやりとり、Webサイトの更新など...）

# 終わりに
私がGoを初めて触ったのは、柴田さん([@yoshiki_shibata](https://twitter.com/yoshiki_shibata))が行なった社内Go研修だった。  

- 第1期Go言語研修が終了しました
    https://yshibata.blog.ss-blog.jp/2016-10-21

バージョン的にはGo1.6が出たあたりで、まだ`context.Context`はなかった。  
[プログラミング言語Go](https://www.amazon.co.jp/dp/4621300253)の練習問題をほぼ全部書いて、チャットとかFTPサーバの実装をしていた。  
私はもともとCからプログラミングを始めていて、「シンプルな並行処理、必要十分な標準パッケージ、これがAlternative Cだ！！！」とすごい感動した（C++はCとは書き味が全く異なる気持ち）。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/gopl" data-iframely-url="//cdn.iframe.ly/phoTIuW"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


その後Goを業務で書きたくて転職して、2018年くらいから業務でもGoを書いている。  
Goを覚えた後C#、Ruby、React、PHP、Pythonも始めたのだけれど、今でも一番Goが好きだ。  
私はあまりコードでGoに貢献できないので、golang.tokyoやGo Conferenceの活動でGoコミュニティに貢献している。  
tenntennさんの発表の通り、今後もGoコミュニティが活発になり、いろいろな人、企業がGoを書くようになると嬉しい。


# 参考
- [Hey! Ho! Let's Go! | Google Open Source Blog](https://opensource.googleblog.com/2009/11/hey-ho-lets-go.html)
- [Contracts — Draft Design](https://go.googlesource.com/proposal/+/master/design/go2draft-contracts.md)
- [Why Generics? | Go Blog](https://blog.golang.org/why-generics)
- [go.dev](https://learn.go.dev/)
- [[発表資料] 今改めて読み直したい Go基礎情報 その1 #golangtokyo](/2019/06/20/golangtokyo25-read-again-awesome-go-article/)
- [HTTP(S) リクエストの発行 | Go1.9 の App Engine スタンダード環境](https://cloud.google.com/appengine/docs/standard/go/issue-requests)
- [eureka Advent Calendar 2019](https://qiita.com/advent-calendar/2019/eureka)
- [[書評] UNIXという考え方―その設計思想と哲学 を読んで「単純さ」の大切さを改めて学んだ](https://budougumi0617.github.io/2019/06/30/review-the-unix-philosophy/)
- [go getで取得されるコードはmasterブランチ(HEAD)がデフォルトではない #golang](https://budougumi0617.github.io/2018/05/10/go-get-from-go1-tag-or-branch/)
- [Go Programming Language Resources](http://go-lang.cat-v.org/)
- [第1期Go言語研修が終了しました](https://yshibata.blog.ss-blog.jp/2016-10-21)

