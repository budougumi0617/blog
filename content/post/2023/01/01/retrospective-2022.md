+++
title= "2022年振り返り"
date= 2023-01-01T03:18:40+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["esssay"]
tags = ["Retrospective"]
keywords = ["振り返り"]
twitterImage = "twittercard.png"
+++

2022年を振り返った。  
例年GitHub上の振り返りもしていたが、2022年はこの記事だけにしようと思う。  
// 2022年中の公開は間に合わなかった。

<!--more-->

# 主な出来事
## 執筆した書籍が全国販売された
- [「詳解Go言語Webアプリケーション開発」という本を発売しました](/2022/07/22/release_go_web_application_book/)
<iframe sandbox="allow-popups allow-scripts allow-modals allow-forms allow-same-origin" style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B0B62K55SL&linkId=7e15e40102e3c13bea0c6f5802ad9c6e"></iframe>

Goの書籍の執筆依頼を受けていて、2022年の夏に無事発売することができた。  
執筆が全然間に合っておらず、最後かなり駆け足になってしまったがきちんと発売できてよかった。  
発売から半年くらい経ったが、[出版社の週間売れ筋ランキング](http://blog.livedoor.jp/lora1/archives/cat_10054792.html)では2位をキープしていたり、Amazonレビューも4.5前後をキープできているのでよいアウトプットができたのかなと思っている。  
近所のイオンモールの本屋さんなどでも物理本が並んでいたので親にも良い顔ができた。  
出版から半年経ったのでひとつだけ弁明しておくと、ハンズオンが始まる後半の原稿はスケジュールの都合上レビューアにレビューしてもらうことができなかった。なので後半のタイポや記述のミスについて、レビューア陣は一切関係ない。  
正誤表も後半部分への指摘が多く、レビューが如何に大事が身にしみた。

また、書き足りないトピックについて補遺的にZenn bookを公開しようと思っていたけれど全然その時間をとれなかった。

### ブログ
2022年はブログを8記事執筆した。  
年間PV数は2022/12/30現在で140,600PVくらいだった。2022年1番人気だった記事は当然ながら出版報告の記事。

- [「詳解Go言語Webアプリケーション開発」という本を発売しました](/2022/07/22/release_go_web_application_book/)


### 寄稿
会社のブログには6記事ほど執筆・コメントを寄稿した。  
「ブログとしてのコンテンツ」というよりはイベント登壇報告ばかりだったけれど、イベント登壇のほうがカロリーは高いので良しとしている。

- [Go Conference 2022 Spring Onlineにシルバースポンサーで協賛・2名登壇します](https://devblog.thebase.in/entry/gocon2022s)
- [New Relic One のCircleCI Integrationでデプロイ頻度やジョブの状態を計測する](https://devblog.thebase.in/entry/2022/04/25/180000)
- [＼非公式／ Go Conference 2022 Spring スポンサー企業4社 アフタートークに2名登壇しました](https://devblog.thebase.in/entry/gocon2022_4sponsor)
- [「新規事業プロダクト開発時の技術選定どうやった？」にBASE BANKチームが登壇しました](https://devblog.thebase.in/entry/how-to-decide-technology-selection-for-startup)
- [ISUCON 12予選に8名（6チーム）が参加しました](https://devblog.thebase.in/entry/isucon12q)
- [「Tech Meetup 〜Goで作る決済サービス〜」にBASE BANK Sectionのメンバーが登壇しました](https://devblog.thebase.in/entry/go_tech_meetup)

## 登壇
昨年は育児の都合で登壇を控えていたけれど、2022年の前半は頑張ることにした。  

### Go Conference 2022 Spring
https://gocon.jp/2022spring/sessions/a10-s/
<div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.1972%;"><iframe src="https://speakerdeck.com/player/5ce760a80d224a869ecb6b8c73959a94" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

Go Conferenceにプロポーザルを出してテストについて発表した。
実務で得た知見をアウトプットできてよかった。

### ＼非公式／ Go Conference 2022 Spring スポンサー企業4社 アフタートーク
https://andpad.connpass.com/event/243953/

<div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.1972%;"><iframe src="https://speakerdeck.com/player/510003a1903d46afa205b417c137ffa5" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

Showcase Gigさん、LayerXさん、ANDPADさんらとの発表はなかなかエキサイティングだった。  
パネルディスカッションではもっと技術力磨かないとなという気持ちにさせられた。

### 新規事業プロダクト開発時の技術選定どうやった？
https://techplay.jp/event/860202
<div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.1972%;"><iframe src="https://speakerdeck.com/player/3f8a3677da664fe9b0d76ef1b62be0fe" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

なぜ自分たちはこの技術を使うのか？という点を改めて考えられたのはよかった。  
またプロダクト志向であるためには技術志向でなければいけないなと思考を整理できたのはよかった。  
プロダクト志向であり続けるには技術に囚われてはいけないが、技術を探求できていないと手持ちの手札に思考が限定されてしまう。  
盲目的にRDBMSを選択してしまうとか、非同期処理を書くのにメッセージキューを絡めたアーキテクチャを選択肢に入れられないとか。それこそ自分でいうとNewSQL、Kafka、データ基盤的なところに全然知見がないのでテックリードなりアーキテクト的なロールで成果を出し続けるためには引き続き研鑽が必要だなと感じている。

## OSS活動
OSSとは言えないが書籍のサンプルコードリポジトリを公開した。90star以上もらえており、Amazonレビューなどと同じくらい嬉しい。
書籍はGitHub Discussionsを使って問い合わせを受けたり、リポジトリに正誤表を公開している。普段使っているツールなので私的には非常にやりやすい。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/go_todo_app" data-iframely-url="//iframely.net/bYCjhBW?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

## ISUCON12に参加した
https://github.com/budougumi0617/isucon12q

[ISUCON12](https://isucon.net/archives/56571716.html)に出場したが惨敗した。  
ISUCON10やISUCON11と異なる構成にテンパってほとんど何もできなかったし、データも壊して大幅にタイムロスしてしまった。  
かなり悔しい結果になったので来年もあるのならばぜひリベンジしたい。

## 続・育児はたいへん
2歳の双子を育てている。保育園が始まればある程度育児に関する時間が少なくなると思っていたのだが、それは誤りだった。  
保育園の送迎や寝かしつけなどもやるので、8時間労働と育児家事を両立しようとすると朝6時半くらいに起きて27時くらいに寝る生活がずっと続いている。  
登園からなかなかしんどくてまあ幼児二人を自転車なり車に乗せるためにはなんかいろいろ格闘するし時間どおり登園できない。
「もう有給とってこのまま車で海とか行くかな」みたいな気持ちに週イチくらいでなっている。
私は裁量労働制だから気持ちを切り替えれば始業できるが、これで9時出社厳守みたいな職業だったら完全に病んでいると思う。
執筆もしている時期は完全に生命が削られているなって感じだった。  
2人いるので、保育園から病気をもらってくる頻度も病欠する日数も多く、コロナももらってきてしまった。  
あまり子育てで何がいい悪いとか言わないけけれど、ネントレだけはまじでやったほうがいいと思う[^nentre]。夜中1,2時間時間ができるだけで負荷が全然違う。  

あと「プレイマットがあるから無理か？」と思っていたけれど12月から[ルンバ](https://www.irobot-jp.com/product/j7/)のサブスクを導入してみて非常によい。
なんだかんだ準備がいろいろあるんだけど[^j7pre]、これで朝の掃除から少し開放された感がある。  
ただプレイマットの上の髪の毛はぜんぜんとれないので結局コロコロしてるんだけどもっと最新機種使えば解決するのかな？生活の効率化はもっとやっていきたい。

[^nentre]: 子供が一人で寝られるようにするねんねトレーニング。
[^j7pre]: 登園前に子供が遊んでいたおもちゃを全部片付けて、ドアを全部あけたり照明を全部つけたり

## 車を買った
ずっとレンタカーで済ませていたが、年初にミニバン（在庫処分のステップワゴン）を買った。[わくわくゲート](https://driver-web.jp/articles/detail/39386)めちゃくちゃ便利なんだけどなんで新型にはないんだろう。
車の中でおむつ交換したり軽食を食べさせたりもするので、幼児二人いたらミニバンじゃなかったらしんどかったと思う。  
いまでは保育園の送迎に使ったり、（休日は自分の時間がないので）さっと始業前にビールを買いに行ったり、その日の思いつきで海を見に行ったりかなり行動が広がった。  
レンタカーが予約できず日程を調整したりすることもあったし、「今日は雨だから車で室内プレイランド行こうか」みたいな「雑に車で出かける」っていう行動は自家用車でないとできないかなと買ってから改めて思った。  

## タスク管理とか
1年間くらいNotionでタスク管理、個人週報などをやってきたんだけれど、12月くらいにTickTickとObsidian（というかMarkdown）に移行した。
プレインテキスト派だったのでMarkdownにほぼ全部集約できて嬉しい。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://ticktick.com" data-iframely-url="//iframely.net/P8nMqyu?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://obsidian.md/" data-iframely-url="//iframely.net/mp0aWa6?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

Notion自体は好きなんだけれど、1行タスクだけでDBのレコードがガンガン増えていくのが肌にあわなかった。
また、時間報的なのを日報Markdownに書いていたり、ナレッジもMarkdwonに書いていたのでタスク管理だけをNotionでするのに冗長さを感じていた。
TickTickは複数デバイス対応でスケジュールタスクだったり習慣化も管理できたりリッチなtodoistという感じのツール。レポートをMarkdownでエクスポートできるので、それを最後日報Markdownに貼り付ける感じの運用ができたのでハマった。
MarkdownファイルはObisidian経由で編集している。private repositoryで管理していたので、デバイス間同期的なのをあきらめていたのだけど、[Self-hosted LiveSync](https://github.com/vrtmrz/obsidian-livesync)プラグインを使うとそれも解決してiPhoneからも同期操作できるようになった。

## その他のプライベート
- リモートワーク
    - 相変わらずリモートワークだった。最近は隔週出社したりしている
    - 通勤時間相当の時間で家事をしているのでリモートワークは引き続き継続できるといいなあ
- 酒
    - だいぶ落ち着いてきたので週末は1日1本飲んでいる
    - うちゅうビールが買える近所のお店を見つけたのと、家の近くのイオンリカールがクラフトビールを大量に置くようになったので捗っている
- 英語
    - 毎日10分読書をしていたが、執筆で生命が削られているときにやめてしまって習慣化も終わった
    - オライリーのサブスクをまだ契約しているのでちゃんと有効活用したい
- 読書
    - あいかわらずKindle読み上げで読書時間を稼いでいるので、図やコードの入る技術書が積ん読になっている
    - 同様の理由でオライリー本の進捗がまったくない。KindleでePub読めるようになったし改善していきたい。
- ガンダム
    - どこかで作りたいなと思ってガンプラ買い始めたんだけど品薄すぎてびっくりしている
    - 逃げたら一つ、進めば二つ。水星の魔女おもしろい
- Apple
    - 個人PCを5,6年ぶりくらいに新調した。MBA M2にしたんだけれどキーボードの質感など非常によい。
    - Paidy使って買ったんだけれど体験がすごかった。これはみんなBNPLやりたくなるのもわかるって感じだった。

# 2022年触った技術
採用なり1on1をするようになってあまり手を動かす時間がなかった。

## Go
New RelicのGo SDKでできないことをラッパーを書いていろいろ対応していた。
具体的には社内共通ライブラリで`COMMIT`の実行時間やMySQLのクエリ情報など取れるようにしている。

## Vue2
そこそこの規模の機能開発を一人でやったり、マイグレーションを担当したりしてけっこう触った。  
ただクラスコンポーネント形式で実装していたのでComposision APIなどトレンドに追従していくにはアンラーンが必要そう？

# 2021年のTryの振り返り
- [2021年振り返り](/2021/12/30/retrospective-2021/)

2022年のKPTを考える前に2021年のTryにちゃんと取り組めたか確認しておく。

|評価|内容|詳細|
|---|---|---|
|△|公私両立| 正直持続性がある生活ではない。
|○|出版!!!!!!|出せた。
|○|フロントエンドの技術習得|ひととおりレビューできるくらいにはなった。
|△|チームの文化をつくる|まだ道半ば
|✗|毎月5冊本を読む|平均3冊くらいしか読んでいない|
|✗|英語の技術書を2か月に1冊読む|英語自体を読めていない
|✗|ISUCON予選突破| 単純なスキル不足を痛感した
|✗|デイキャンプにいく|なかなか幼児二人は難しい

子供が1歳すぎたらプライベートの時間が復活すると思っていたがそんなことはなかった。育児って厳しい。

# 2022年のまとめ（KPT）

## Keep
- 無事に書籍を出版できた
- 採用に関して結構コミットできた
- 1年間育児家事と仕事を両立した
- なんだかんだ一定の読書量はキープできた

## Problem
- 出版できたがタイポなどが多かった
- 手を動かす時間が少なくなってしまった
- 積ん読が増加傾向にある
    - とくに技術書が全然読めなかった
- 睡眠不足
    - 感情的になったりやっぱり後天的にショートスリーパーになるのは無理

「子供がいるからしょうがないよね？」と「30代後半妥協しつづけていいんか？」という気持ちが常に揺らいでいる1年だったんだけど、年末は[限りある時間の使い方](https://amzn.to/3WTQK1l)と[「逃げたら一つ、進めば二つ」](https://g-witch.net/music/novel/)がだいぶ刺さっている。

## Try
- 個人活動時間を作る
    - 深夜しか時間が取れていないのでここを解決しないと他が解決できない
    - 子供連れてカフェなどに行く？とも思うけれど、年少以下の幼児2人つれていくのはシンプルにキツい
- 日々の生活をもっと自動化・効率化していく
    - 家事もそうだけど、週の振り返りするにも結構時間がかかっているのでもっとテンプレートなり自動生成をできるようにして少しでも限られた時間を有効に使いたい
- 技術書の積ん読を減らす
- 睡眠時間を増やす
- iPadを使いこなす
    - 新春セールでiPad Air買いたいなと思っている
    - 手書きによるアイデア出しや、令和の勉強法的なのを理解する的な意味でApple Pencile駆使したタブレット操作を活用していきたい
- 2023年こそデイキャンプに行く
- 執筆リベンジ
    - もう一冊書いてみない？とお話をいただいたのでもう1回頑張ってみようかなと思っている
    - まだ構想も何もできていない

# 2022年の記録

以下、もろもろの記録。

## ブログなどのKPI
毎週Twitterのフォロワー数とブログのスコアを集計している[^kpi_issue]。

[^kpi_issue]: twitterのフォロワー数はSocialDogで取るようになった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/blog-kpi-collector" data-iframely-url="//iframely.net/13K41P4?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

2022年は以下のような数値だった。

|集計日|Twitter フォロワー数|ブログの週間PV|ブログの累計はてブ数|
|---|---|---|---|
|2021/12/26|1204|2577|2910
|2022/03/27|1230|2197|2929
|2022/06/26|1287|2722|2936
|2022/09/25|1494|2567|3087
|2022/12/25|1540|2440|3094

Twitterのフォロワーが1,500人を超えた。  
書籍を出版したことでフォロワーが結構増えた。Twitterは2022年買収などでいろいろあったけれどあまり別のプラットフォームに移動することは考えていない。

## 参加した技術イベント
2022年は自分が登壇していないイベントには参加できていない。

## 登壇したイベント
如何に時間をつくるかが問題だったけれど登壇してよかった。

- [Go Conference 2022 Spring](https://gocon.jp/2022spring/)
    - [Go Conferrence 2022 Springで登壇した #gocon](/2022/04/25/gocon2022spring/)
- [＼非公式／ Go Conference 2022 Spring スポンサー企業4社 アフタートーク](https://andpad.connpass.com/event/243953/)
- [新規事業プロダクト開発時の技術選定どうやった？-カケハシ×LINE×BASEの開発者が振り返る技術選定プロセスと内省-](https://techplay.jp/event/860202)
- [Tech Meetup 〜Goで作る決済サービス〜](https://upsider.connpass.com/event/254313/)


## 2022年に読んだ書籍
- https://booklog.jp/users/budougumi/stats

41冊読み終わった。「エクストリームプログラミング」や「良い戦略、悪い戦略」のような未読の名著を読んだけどやはり良いものはよかった。  
前述したとおりオライリー本を買っただけで全然読めていないのでちゃんと消化したい。

- [ユニコーン企業のひみつ ―Spotifyで学んだソフトウェアづくりと働き方](https://amzn.to/3WSqLal)
    - [[書評]ユニコーン企業のひみつ](/2022/01/07/review_competing_with_unicorns/)
- [NO RULES(ノー・ルールズ) 世界一「自由」な会社、NETFLIX](https://amzn.to/3GuObxi)
    - [[書評] NO RULES 世界一「自由」な会社、NETFLIX | コントロールではなくコンテキストを](/2022/02/16/review_netflix_no_rules/)
- [THE CULTURE CODE 最強チームをつくる方法](https://amzn.to/3i2VtyV)
    - [[書評]THE CULTURE CODE 最強チームをつくる方法 | 何が良いチームをつくるのか？](/2022/02/27/review_culture_code/)
- [人に頼む技術コロンビア大学の嫌な顔されずに人を動かす科学](https://amzn.to/3Gu8gUt)
    - [booklogの感想メモ](https://booklog.jp/users/budougumi/archives/1/B07V27Y42J)
- [「技術書」の読書術 達人が教える選び方・読み方・情報発信&共有のコツとテクニック](https://amzn.to/3WxRc5u)
    - [booklogの感想メモ](https://booklog.jp/users/budougumi/archives/1/B0BF469YLK)
- [村上式シンプル英語勉強法](https://amzn.to/3YYO44a)
    - [booklogの感想メモ](https://booklog.jp/users/budougumi/archives/1/B0081MAET6)
- [Fearless Change アジャイルに効く アイデアを組織に広めるための48のパターン](https://amzn.to/3i1x5xw)
    - [booklogの感想メモ](https://booklog.jp/users/budougumi/archives/1/462108786X)
- [ライト、ついてますか 問題発見の人間学](https://amzn.to/3Gr6yDk)
    - [booklogの感想メモ](https://booklog.jp/users/budougumi/archives/1/B09BDWRCXN)
- [プロを目指す人のためのTypeScript入門 安全なコードの書き方から高度な型の使い方まで](https://amzn.to/3Q0z4Pr)
    - [booklogの感想メモ](https://booklog.jp/users/budougumi/archives/1/B09Y527YPV)
- [「読む」だけで終わりにしない読書術 1万冊を読んでわかった本当に人生を変える方法](https://amzn.to/3WAg3pr)
    - [booklogの感想メモ](https://booklog.jp/users/budougumi/archives/1/B09MCXV7XB)
- [ジェームズ・クリアー式 複利で伸びる1つの習慣](https://amzn.to/3G4fH3e)
    - [booklogの感想メモ](https://booklog.jp/users/budougumi/archives/1/B07YY2WV6K)
- [EMPOWERED 普通のチームが並外れた製品を生み出すプロダクトリーダーシップ](https://amzn.to/3GxmxzV)
- [小さなチーム、大きな仕事　働き方の新しいスタンダード](https://amzn.to/3VBr6NM)
- [国際エグゼクティブコーチが教える 人、組織が劇的に変わる ポジティブフィードバック](https://amzn.to/3Z3Apck)
- [「Why型思考」が仕事を変える 鋭いアウトプットを出せる人の「頭の使い方」](https://amzn.to/3Cgfh8N)
- [楽々ERDレッスン](https://amzn.to/3jM7sBe)
- [教養としての決済](https://amzn.to/3VHd7Wx)
- [Fearless Change アジャイルに効く アイデアを組織に広めるための48のパターン](https://amzn.to/3QlcocV)
- [もしアドラーが上司だったら](https://amzn.to/3WSSB6s)
- [Obsidianノート術: ライフログや個人のノートはMarkdownでローカルに保存](https://amzn.to/3vrG8er)
- [ＯＯＤＡ　ＬＯＯＰ（ウーダループ）―次世代の最強組織に進化する意思決定スキル](https://amzn.to/3CaYIes)
- [リーダーの作法 ―ささいなことをていねいに](https://amzn.to/3i54DLi)
- [みんなでアジャイル ―変化に対応できる顧客中心組織のつくりかた](https://amzn.to/3vqWzaS)
- [達人が教えるWebパフォーマンスチューニング　〜ISUCONから学ぶ高速化の実践](https://amzn.to/3WDYKUl)
- [マッキンゼーで叩き込まれた 超速フレームワーク―――仕事のスピードと質を上げる最強ツール](https://amzn.to/3WS50Yo)
- [エクストリームプログラミング](https://amzn.to/3GrR015)
- [問いかける技術 ― 確かな人間関係と優れた組織をつくる](https://amzn.to/3jJG5I4)
- [良い戦略、悪い戦略](https://amzn.to/3WE5IbV)
- [情報をまとめて・並べるだけ！超シンプルな「手帳」兼「アイデア帳」運用術: 文章を書き、考える人のためのObsidian活用術 情報整理大全](https://amzn.to/3VwWvRj)
- [知的トレーニングの技術〔完全独習版〕](https://amzn.to/3IbLyli)
- [Docs for Developers: An Engineer’s Field Guide to Technical Writing](https://amzn.to/3jx6Cby)
- [チーズはどこへ消えた？](https://amzn.to/3vNomCF)

## 個人ブログ
全然かけなかった。時間がないのもそうだけれどあまり手を動かさなくなったっていうのもあるかもしれない。

- [[書評]ユニコーン企業のひみつ](/2022/01/07/review_competing_with_unicorns/)
- [[Notion] Mac Appでフォントのサイズが大きくならない](/2022/01/30/notion_size_up_font_on_mac_app/)
- [[書評] NO RULES 世界一「自由」な会社、NETFLIX | コントロールではなくコンテキストを](/2022/02/16/review_netflix_no_rules/)
- [[書評]THE CULTURE CODE 最強チームをつくる方法 | 何が良いチームをつくるのか？](/2022/02/27/review_culture_code/)
- [[New Relic] FACET CASE句を使って外部サービスのエンドポイントのレスポンスタイムを集計する](/2022/03/22/newrelic_make_external_api_by_facet_case_capture/)
- [Go Conferrence 2022 Springで登壇した #gocon](/2022/04/25/gocon2022spring/)
- [「詳解Go言語Webアプリケーション開発」という本を発売しました](/2022/07/22/release_go_web_application_book/)
- [GitHub CLIを使って任意の条件のissueを一括closeする（一括操作する）](/2022/09/20/closed-stale-github-issue-by-manual/)

# 関連
- [2021年振り返り](/2021/12/26/retrospective-2021/)
- [2021年振り返り(GitHub編)](/2021/12/26/retrospective-on-github-2021/)
- [2020年振り返り](/2020/12/27/retrospective-2020/)
- [2020年振り返り(GitHub編)](/2020/12/15/retrospective-on-github-2020/)
- [2019年振り返り](/2019/12/22/retrospective-2019/)
- [2019年振り返り(GitHub編)](/2019/12/14/retrospective-on-github-2019/)
- [2018年振り返り](/2018/12/27/retrospective-2018/)
- [2018年振り返り(GitHub編)](/2018/12/15/retrospective-on-github/)
- [2017年振り返り](/2017/12/30/retrospective-2017/)
- [2017年振り返り(GitHub編)](/2017/12/27/retrospective-on-github/)

