+++
title= "2018年振り返り"
date = 2018-12-27T07:34:49+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["esssay"]
tags = ["Retrospective"]
twitterImage = "twittercard.png"
+++


2018年を振り返ってみる。GitHubベースの振り返りは別記事にまとめた。

![冬に登った野伏ヶ岳](/2018/12/27_nobuse.jpg)

- [2018年振り返り(GitHub編)](/2018/12/15/retrospective-on-github/)

<!--more-->

# 主な出来事

## 技術書典5で技術同人誌を販売した
- [ゴーファーの書 | 技術書典5](https://techbookfest.org/event/tbf05/circle/37230005)

golang.tokyo有志の合同紙に混ぜてもらって数ページ執筆した。私の執筆部分がどれくらい寄与しているかわからないが、多数の方々に手にとっていただけたので非常に嬉しい。
ブログとは違い、ある程度のボリュームをなるべく周辺知識をフォローしながらストーリー立てて書くのは良い経験になった。

![技術書典5](/2018/12/27_gopher.jpg)

## 社外勉強会で登壇を7回した
今年は夏くらいから「毎月1回登壇する」を目標に活動していた。技術書典5があった10月以外はちゃんとできた気がする。
10分枠くらいならそれなりに発表できるようになれた。来年はCFP出すような発表枠で登壇できるようになりたい。

## 毎週ブログを書くようになった。1年間で81記事書いた。
[昨年の振り返り](/2017/12/30/retrospective-2017/)で述べたとおり、今年はちゃんとアウトプットをしようとブログを書き続けた。
最初は月間1,000PVもいかなかったが、いまでは9,000PVくらいあるのでそれなりの質を保ちながら書けているかなと思っている。
ブログについてはしっかり習慣化できたので継続していく。ブログメンタリングも受けてよかった。会社の公式ブログにも寄稿できた。

- [@kakakakakku さんのブログメンタリングを3ヶ月間受けて卒業した](/2018/07/10/graduate-blog-mentoring-by-kakakakakku/)
- [Goでスタックトレースを構造化して取り扱う | Freee Developers Blog](https://developers.freee.co.jp/entry/2018/12/23/213000)

## golang.tokyoの勉強会(社外勉強会)を勤務先で実施、登壇できた
- [golang.tokyo #17 今あらためてテストの話](https://golangtokyo.connpass.com/event/96200/)

golang.tokyoは運営メンバーの会社施設を持ち回りして勉強会を開催しているので、私の勤務先でも勉強会を開催した。Goの勉強会を開催するような会社に勤めたいと思っていたので、主催できてよかった。主催者枠で20分の発表もすることができた。

- Tour of testing in 2018
  - https://speakerdeck.com/budougumi0617/tour-of-testing-in-2018

## OSSとしてCLIツールを公開した。
- https://github.com/budougumi0617/lsas

今年はCLIコマンドを一つ作った。「何かネタないかな？」と思っていたら社内のSREから「AWS CLIがめんどくさい」と聞いたので作った。このOSSは独りよがりじゃなくて、SREの人に「何が必要なのか？」「どう作っておけばパイプしやすいのか？」などをヒアリングして作った。ちょっとした顧客要求の分析、要件定義までできたのでよい経験になった。結構社内で多用されているので単純に作った甲斐があって嬉しい。また、社内のエンジニアからだがPRをもらったりもできてOSS感がでている。

## golang.tokyo運営に参加した
春からgolang.tokyoという勉強会コミュニティの運営に参加した。私はあまりコーディングスキルがないが、コミュニティのお手伝いならGoにコミットできるかなと思って参加している。他社のGopherさんたちにいろいろ教えてもらったり自分自身の成長の時間にもなっているので引き続き参加していく。

## その他のプライベート
- ボルダリングを継続できた
  - ほぼほぼ毎週通えている
- 引っ越しをした
  - 12月に社会人になってからずっと住んでいた武蔵小杉を離れて立川に引っ越した。中央線というより山手線通勤にまだ慣れていない…
- テント泊登山・スノーシュー登山をした
  - 今年はあまり山に行けなかったので来年はもう少し登山したい
  - ![nobse](/2018/12/27_nobuse.jpg)
  - ![camp](/2018/12/27_camp.jpg)
- ポーカーをやっている
  - 大学の友達や社内の知人とポーカーをしている。あまり強くはないがテキサスホールデムは本当に奥が深い。
- あいかわらずPerfumeが好き
  - 今年は3回ライブに行けた（3回目は大晦日なので正確にはまだ2回しか行ってない）。
- 自作キーボードを始めた
  - ずっとHHKBだったのだが、今は以下のキーボードにした。ホームポジションがほとんど崩れないし今年のベストバイ。
  - [Corne Cherry](https://booth.pm/ja/items/869375)
  - ![corne](/2018/12/27_corne.jpg)

# 今年触った技術
## Go
- https://golang.org/

今年の春からは業務でもGoを触るようになった。Railsの裏のマイクロサービスをGoで作っている。本体側を触ることも多いので、Rubyなども触っているが、メインの業務がGoになったので非常に嬉しい。
やはり業務で扱うと書籍での学習やチュートリアルを少しいじっただけでは得られない経験が出来るので非常によい。
逆にいうとまだまだひよっこGopherなのでもっとコード書かないといけない。

- [Go関連記事](/categories/go/)

## gRPC
- https://grpc.io/

開発しているプロダクトで利用している。アサイン前に少し予習していたので、すんなり入っていけた。
（OpenAPIなどを使わない）REST APIの設計に比べるとだいぶラクな気がするが、やはり運用次第なんだなというところは痛感してる。
マイグレーション時のお作法などは個人ではあまり学べないので業務で利用できてよかった。

- [gRPC関連記事](/categories/grpc/)

## JavaScript(React.js)
- https://reactjs.org/

JavaScript自体を触るのが初めてだったので新しい道を開けた。
はっきり言って動的型付け言語に強い苦手意識を持っていた。とくにJavaScript周りは進歩も激しすぎるので敬遠していた。
ただ、業務で触ることになりやってみると、周辺エコシステムのおかげでだいぶ気持ちよく開発ができた。

- FlowTypeで型の恩恵を受けることができる
- create-react-appでpcakge.jsonと戦わなくて済む
- JestでTDDしながら開発できる
- ホットリロードとChrome Dev ToolsでREPLを使って実装できる

来年は「TypeScriptに挑戦するぞ！」と思えるくらいフロントエンド技術に興味が持てるようになったのは自分の中で今年一番の変化かもしれない。

- [React.js関連記事](/tags/react/)

## Elixer
- https://elixir-lang.org/

勉強会で写経したレベルだが関数型言語に初めて挑戦した。読了後は触れていないが、設計の仕方やアプローチの幅が少し広がった気がする。

## Spinnaker
- https://www.spinnaker.io/

Goの勉強会はメルカリさんがよく発表されていて、その中で頻出していたので自分でも少し触ってみていた。
GKE上に構築して、カナリーリリースについても学べたのでクラウドネイティブな継続的デプロイについて手を動かしながら知ることができた。
年の前半に個人的に触っただけなので、来年業務のどこかで使えないか虎視眈々としている。

- [Spinnaker関連記事](/tags/spinnaker/)

# 2017年のTryの振り返り
- [2017年振り返り](/2017/12/30/retrospective-2017/)

2017年の振り返りでは、以下の項目を振り返りにあげていた。

- もっとコードでOSSに貢献する
- 自分のOSSをもっと作る
- GoとGCPでWebアプリを作る
- クラウド・コンテナ技術の知識を身につける
- 朝の学習時間をもっと確保・習慣化する
- ブログをもっと更新する

ブログの更新はできたが、他の目標はあまりできなかった。朝活は定期的にできていたが、習慣化まではできなかった。
きちんと動く自作OSSはひとつしかできなかったし、OSSにいくつかPRは出せたが、どれもタイポの修正などだった。

# 2018年のまとめ(KPT)

## Keep
- 初めて社外勉強会で登壇できた
- Goのコミュニティ活動に参加するようになった
- 業務でGoを書くようになった。20分の社外発表もできた
- 業務でReact.jsを使えるようになった。社外発表もできた
- （合同誌だが、）本を執筆して技術書典で発売できた
- 毎週ブログを更新できた。
- （`--amend`でちょっとズルもしているが）1年間GitHubにコミットし続けた。
  - https://github.com/budougumi0617?tab=overview&from=2018-01-01&to=2018-12-31


## Problem
- OSSにコーディングでプルリク出せるレベルではない
  - 去年からの継続Problem...
- 読み終わってない本が多い。読書量が少ない
  - 読了したらちゃんと（知識の確認と忘れないために）書評も書くこともセットにする
  - 電車通勤が長くなったので通勤時間を有効活用したい。
- 生産性を上げる
  - これも去年からの継続Problem
  - もっとコーディング量を上げないと質がついてこない（経験値がたまらない）
- 相変わらず英語が苦手
  - インプットが少なくなる、または公式サイトを直接見れない

## Try
- 2ヶ月にひとつのペースで自分のOSSを他人が使える状態で公開する
- 3月までにGoとGAE+DatastoreでWebアプリを作る
- 8月までにEC2+RDS or LambdaでWebアプリを作る
- 8月までにTypeScriptを覚える
  - NetlifyかFirestoreで何か公開する
- 1月から朝活する曜日を作る
  - 五反田へ通勤ラッシュ前に出社したい
- 英語の勉強時間を作る
  - Kindleでなかなか勉強できないので頑張って物理本を持ち歩く
- 一人称をそろそろ「私」にしたい
  - 職場ではいつも「ぼく」って言っていたがそろそろ直したい

# 2018年の記録

以下、勉強会や読書の記録。

## 参加した技術イベント
- [golang.tokyo #12](https://golangtokyo.connpass.com/event/76540/)
  - [[Go]golang.tokyo #12 参加メモ #golangtokyo](/2018/01/31/golangtokyo-12/)
- [Redash Meetup #0.1 - Redash 初心者向けハンズオン](https://redash-meetup.connpass.com/event/75423/)
  - [Redash Meetup #0.1 - Redash 初心者向けハンズオン #redashmeetup](/2018/01/31/redashmeetup-hands-on/)
- [Kubernetesで実現する高可用性システム -- cndjp第3回](https://cnd.connpass.com/event/75617/)
  - [[k8s]Cloud Native Developers JP 第3回勉強会 #cndjp3 参加メモ](/2018/02/03/kubernetes-with-availability-cndjp3/)
- [GCPUG Tokyo Container Builder Day February 2018](https://gcpug-tokyo.connpass.com/event/75961/)
  - [GCPUG Tokyo Container Builder Day February 2018 #gcpug 参加メモ](/2018/02/05/gcpug-container-builder-day/)
- [Go 1.10 Release Party in Tokyo](https://gocon.connpass.com/event/78128/)
  - [Go 1.10 Release Party in Tokyo参加メモ #go110party](/2018/02/22/go110party/)
- [Google Cloud Platform hands-on (Cloud Study Jam) #CloudStudyJam](https://www.meetup.com/ja-JP/gdg-tokyo-jp/events/248338420/)
  - [GCPハンズオン( #CloudStudyJam )に参加した](/2018/03/05/gcp-cloud-study-jam/)
- [golang.tokyo #13](https://golangtokyo.connpass.com/event/79039/)
  - [[Go]golang.tokyo #13 - Macでハンズオンアプリを動かすまでに必要だったこと #golangtokyo](/2018/03/14/report-golang-tokyo-13-android-with-gomobile/)
- [Kubernetes Network Deep Dive!（Istioもあるよ）- cndjp第4回](https://cnd.connpass.com/event/79999/)
  - [Kubernetes Network Deep Dive!（Istioもあるよ） cndjp第四回参加メモ #cndjp4](/2018/04/01/kubernetes-network-deep-dive-cndjp4/)
- [Go Conference 2018 Spring](https://gocon.connpass.com/event/82515/)
  - [Go Conference 2018 Spring 参加メモ #gocon](/2018/04/17/go-conference-2018-spring/)
- [golang.tokyo #14](https://golangtokyo.connpass.com/event/82723/)
  - [golang.tokyo #14 ゴールーチンとチャネルによる並行処理 参加メモ #golangtokyo](/2018/04/19/golangtokyo-14-goroutine/)
- [GCPUG Tokyo Kubernetes Engine Day April 2018](https://gcpug-tokyo.connpass.com/event/81224/)
  - [[k8s]GCPUG Tokyo Kubernetes Engine Day April 2018参加メモ #gcpug](/2018/04/28/gcpug-tokyo-kubernetes-engine-day-april-2018/)
- [あつまれ！ CI/CDツール大集合！ - cndjp第5回](https://cnd.connpass.com/event/84310/)
  - [[k8s]あつまれ！ CI/CDツール大集合！ - cndjp第5回 参加メモ #cndjp5](/2018/05/02/cndjp5-cicd-tools/)
  - [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)
- [隅からスミまでKubernetes総復習 - cndjpシーズン1まとめ](https://cnd.connpass.com/event/89854/)
  - [[k8s]隅からスミまでKubernetes総復習 - cndjpシーズン1まとめ参加メモ #cndjp #k8s](/2018/06/15/cndjp6-review-kubernetes/)
- [mercari.go #1](https://mercari.connpass.com/event/91306/)
  - [mercari.go #1 参加メモ #mercarigo](/2018/07/05/report-mercarigo-1/)
- [GCPUG Tokyo gVisor Day July 2018](https://gcpug-tokyo.connpass.com/event/90909/)
  - [GCPUG Tokyo gVisor Day July 2018 参加メモ #gcpug #gvisor](/2018/07/18/report-gcpug-gvisor/)
- [golang.tokyo #16](https://golangtokyo.connpass.com/event/92225/)
- [Meetup in Tokyo #40 -Kubernetes-](https://line.connpass.com/event/92049/)
  - [LINE Developer Meetup in Tokyo #40 -Kubernetes-参加メモ #LINE_DM #k8s](/2018/07/20/report-line-developer-meetup-40-kubernetes/)
- [mercari.go #2](https://mercari.connpass.com/event/95665/)
  - [mercari.go #2 参加メモ #mercarigo](/2018/08/12/report-mercarigo2/)
- [golang.tokyo #17](https://golangtokyo.connpass.com/event/96200/)
- [Go 1.11 Release Party in Tokyo](https://gocon.connpass.com/event/95631/)
- [mercari.go #3](https://mercari.connpass.com/event/98213/)
- [golang.tokyo #18](https://golangtokyo.connpass.com/event/101174/)
- [golang.tokyo #19](https://golangtokyo.connpass.com/event/105072/)
- [Kubernetes読書会](https://sea-region.github.com/wantedly/kubernetes-code-reading-party/issues/1)
- [第22回横浜Go読書会](https://yokohama-go-reading.connpass.com/event/104986/)
- [エリック・エヴァンスのドメイン駆動設計 読書会](https://evans-ddd-book-read.connpass.com/)
- [Go Conference 2018 Autumn](https://gocon.connpass.com/event/109368/)
- [golang.tokyo #20](https://golangtokyo.connpass.com/event/111077/)
  - [golang.tokyo #20 忘年LT大会！！ 参加メモ #golangtokyo](/2018/12/19/report-golangtokyo-20/)
- [【シューマイ】もくもく会〜クリスマス？なにそれ、新しいフレームワーク？](https://shuuu-mai.connpass.com/event/111002/)


相変わらずひたすらGoの勉強会に参加してた。年の後半のイベントは勉強会運営だったり、登壇だったりして感想が書けていない。


## 登壇したイベント
- [あつまれ！ CI/CDツール大集合！ - cndjp第5回](https://cnd.connpass.com/event/84310/)
  - [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)
- [Vue.js/React/Go/Rails5.2のリアル★ShuuuMai #04](https://connpass.com/event/86508/)
  - [[発表資料]Go入門](/2018/05/30/go-introduction/)
- [Go(Un)Conference（Goあんこ）LT大会 3kg](https://gounconference.connpass.com/event/92794/)
  - [[発表資料]初級者向けGoの落とし穴と解説 #gounco](/2018/07/26/presentation-gounco-lt3/)
- [WEBエンジニア勉強会 #08](https://web-engineer-meetup.connpass.com/event/94200/)
  - [[発表資料]Spinnakerを利用した自動カナリー分析 #WEBエンジニア勉強会08](/2018/07/27/presentation-web-engineer-study-meeting8/)
- [golang.tokyo #17](https://golangtokyo.connpass.com/event/96200/)
  - [Tour of testing in 2018](https://speakerdeck.com/budougumi0617/tour-of-testing-in-2018)
- [Go(Un)Conference（Goあんこ）LT大会 4kg](https://gounconference.connpass.com/event/99487/)
  - [[発表資料]go-cloudとWireを利用したDI #gounco #go](/2018/10/19/presentation-gounco-lt4/)
- [WEBエンジニア勉強会 #10 (東京都, 渋谷)](https://web-engineer-meetup.connpass.com/event/104523/)
  - [[発表資料] React+Redux 次の一歩 補足資料 #WEBエンジニア勉強会10](/2018/11/09/presentation-web-engineer-meetup-10/)

前述の通り。

## 2018年に読んだ書籍
- [SOFT SKILLS　ソフトウェア開発者の人生マニュアル](https://amzn.to/2EHRW34)
  - [[書評]SOFT SKILLSを読んだ](/2018/01/03/review-soft-skills/)
- [スリープ・レボリューション ](https://amzn.to/2EPpcX2)
  - [[書評]スリープレボリューションを読んだ](/2018/01/10/review-sleep-revolution/)
- [世界のエリートがやっている 最高の休息法](https://amzn.to/2CxwjAW)
  - [[書評]世界のエリートがやっている 最高の休息法を読んだ](/2018/01/16/review-the-best-way-to-rest/)
- [Goならわかるシステムプログラミング ](https://amzn.to/2Cw4LM8)
- [[書評]Goならわかるシステムプログラミングを読んだ](/2018/02/26/review-go-system-programming/)
- [SQL 第2版 ゼロからはじめるデータベース操作](https://amzn.to/2BDupx4)
  - [[書評]SQL 第2版 ゼロからはじめるデータベース操作 を読んだ](/2018/05/23/review-sql-study-database-operation-from-zero/)
- [Continuous Delivery With Spinnaker](https://www.spinnaker.io/ebook/#continuous-delivery-with-spinnaker)
  - [Continuous Delivery With Spinnakerでマイクロサービスの継続的デリバリーの課題と解決手法を学ぶ](/2018/06/14/review-continuous-delivery-with-spinnaker/)
- [マイクロサービスアーキテクチャ](https://amzn.to/2CzlMFa)
  - [[書評]マイクロサービスアーキテクチャを読んだ](/2018/06/26/review-microservices-architecture/)
- [Google Cloud Platform エンタープライズ設計ガイド ](https://amzn.to/2RcrepS)
  - [[書評]Google Cloud Platform エンタープライズ設計ガイドを読んだ](/2018/08/04/rewview-gcp-enterprise-design-guide/)
- [エンジニアの知的生産術 ―効率的に学び、整理し、アウトプットする](https://amzn.to/2CzF5OT)
  - [[書評]エンジニアの知的生産術を読んだ](/2018/10/03/review-the-productive-technique-of-engineer/)
- [Go言語でつくるインタプリタ](https://amzn.to/2EOsWrC)
  - [[書評] Go言語でつくるインタプリタ を読んだ #go](/2018/10/30/review-writing-an-gnterpreter-in-go/)
- [Real World HTTP ―歴史とコードに学ぶインターネットとウェブ技術](http://amzn.to/2pV2Ufy)
- [プログラミングElixir](http://amzn.to/2ClBUuR)
- [現場で役立つシステム設計の原則 〜変更を楽で安全にするオブジェクト指向の実践技法](http://amzn.to/2Cl39ph)
- [プログラマのためのGoogle Cloud Platform入門 サービスの全体像からクラウドネイティブアプリケーション構築まで](http://amzn.to/2CndNfo)

一部は書評を書けていないが、だいたいの本でちゃんと書評を書いた。完全な洋書も一冊読んでちゃんと書評にまとめることができたのは、英語苦手＋コンプレックスの自分にとって少し自信になった。来年はもう少し洋書にも挑戦していきたい。

## 2018年に読み終わらなかった書籍
- [エリック・エヴァンスのドメイン駆動設計](https://amzn.to/2BIceWK)
- [クリーンアーキテクチャ](https://amzn.to/2Rd236F)
- [APIデザインの極意 Java/NetBeansアーキテクト探究ノート](https://amzn.to/2RcrW6w)
- [テスト駆動開発](https://amzn.to/2BEaz4K)
- [Go言語による並行処理](https://amzn.to/2Rl0rYk)
- [Kubernetes完全ガイド](https://amzn.to/2BBKsew)
- [入門 Kubernetes](https://amzn.to/2EPhUCz)
- [Amazon Web Services 定番業務システム12パターン 設計ガイド](https://amzn.to/2Cyl9Ml)
- [Amazon Web Services パターン別構築・運用ガイド 改訂第2版 ](https://amzn.to/2BGqKyw/)
- [SQLパフォーマンス詳解](https://amzn.to/2BDogkf)
- [Effective SQL](https://amzn.to/2CxQz5j)
- [プログラミングRust](https://amzn.to/2BFPZRt)
- [ベタープログラマ](https://amzn.to/2Tapkn6)


設計関係・インフラ周りの本が積読になってしまっている（一部は読了しているが咀嚼しきれていない）。
来年は個人開発などもしていきたいので、読んで得た知識で実際に手を動かしていきたい。

# 関連
- [2018年振り返り(GitHub編)](/2018/12/15/retrospective-on-github/)
- [2017年振り返り](/2017/12/30/retrospective-2017/)
- [2017年振り返り(GitHub編)](/2017/12/27/retrospective-on-github/)
