+++
title= "Go Conference 2018 Spring 参加メモ #gocon"
date= 2018-04-17T22:52:49+09:00
draft = false
slug = ""
categories = ["go","report"]
tags = ["golang", "gocon"]
author = "budougumi0617"
+++

先日行われたgoconの参加メモ。今回は2トラック制だったので、自分が見たものだけ。
公開されているスライド資料などは以下のリンクに添付されている。

**Go Conference 2018 Springイベント資料一覧**  
https://gocon.connpass.com/event/82515/presentation/

`runtime`のコードを見たり、API設計や公式の歩き方、gRPCマイクロサービスにおけるテストの話まで、非常に幅広い技術スタックの知見を得ることができた。
また、学生エンジニア・新卒エンジニアの方々の熱意や技術力を目の当たりにすることも出来て、アラサーエンジニアとしては畏怖と尊敬を覚えた。  
技術的にもパッション的にも(英語的にも…)、もっと技術を磨いていかないとと感じた一日になった。以下、発表メモと若干の覚え書き。

|||
|---|---|
|URL|https://gocon.connpass.com/event/82515/|
|会場|サイボウズ株式会社 東京オフィス (東京日本橋タワー)|
|日時|2018/04/15(日) 09:00 〜 18:00|
|資料|https://gocon.connpass.com/event/82515/presentation/|
|ハッシュタグ|[#gocon](https://twitter.com/hashtag/gocon)|
|Toggeter|https://togetter.com/li/1218219|

# How the Go runtime implements maps efficiently (without generics)
[Dave Cheney](https://twitter.com/davecheney)

GoのcontributorであるDave-sanのキーノート。

正直英語はあまりできないので、皆さんの翻訳ツイートから。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Go の map 実装を generics も boxing もなしで高速に実装する仕組みの解説。コンパイラが宣言された map の型ごとにアルゴリズム構造体を用意して、map 操作を syntax sugar として展開する際にアルゴリズムを渡すことでうまくやっている。なるほど。 <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; ymmt (@ymmt2005) <a href="https://twitter.com/ymmt2005/status/985332433582080001?ref_src=twsrc%5Etfw">2018年4月15日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">keyもvalueもunsafe.Pointer ここ　<a href="https://t.co/3DOwkSYAha">https://t.co/3DOwkSYAha</a> <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; そな太 (@sonatard) <a href="https://twitter.com/sonatard/status/985330183757381632?ref_src=twsrc%5Etfw">2018年4月15日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">type typeAlg struct { hash, equal }ってのが型に対応したハッシュ関数と同値比較の方法を保持する <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; nasa9084@某某某某(0x19歳) (@nasa9084) <a href="https://twitter.com/nasa9084/status/985330892707999745?ref_src=twsrc%5Etfw">2018年4月15日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">なるほどそういう方法にするとmapの実装を１つに出来てバイナリのサイズを小さくできるし速さもある程度両立できるということか. 賢いなぁ <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; おりさの (@orisano) <a href="https://twitter.com/orisano/status/985332113405652994?ref_src=twsrc%5Etfw">2018年4月15日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


mapの実装の話。基本的なHashMapのアルゴリズムから、C++, JavaのMapの実装との比較も解説されていた。

- Goはhashmapの実装に`interface{}`をつかっていない。
- コンパイルするとmap操作は`runtime.mapaccess1`といった関数に展開されている。
- HashMapの実装は`src/runtime/hashmap.go`に全部入っている。
- mapの実装はKey/Valueを`unsafe.Poointer`で持つ。
- じゃあどうやってmap操作を行っているかというと、`maptype`型にもつ情報から値の比較などを行っている。
  - https://golang.org/src/runtime/type.go#L349
- たとえは`map[key]`で値を取得する実装はここ。
  - https://golang.org/src/runtime/hashmap.go#L343
- 実行時にアルゴリズムを渡せるので、型ごとのmapが作られること無くバイナリの誇大化を避けている。
- `channel`や`select`の実装も読んでためになる。
  - https://golang.org/src/runtime/select.go

実際に解説を受けて背景を知ってから`runtime`パッケージを見るの非常におもしろい。  
また標準パッケージを読む楽しさを知ってしまった。  
単純なアルゴリズムこそ最適化のためにいろいろされているんだなと感じた。  

[codehex](https://twitter.com/codehex)さんが[ブログ](https://codehex.hateblo.jp/entry/2018/04/16/114616)に懇親会でDave-sanから聞いた情報も書いてくださっている。

# Go at Cybozu
サイボウズ株式会社 [@ymmt](https://twitter.com/ymmt2005)  
https://speakerdeck.com/ymmt2005/go-at-cybozu

スポンサーセッション。  
今回のgoconはサイボウズさんの会場だったのだが、すごいキレイだった！！  
会場の部屋もイベント規模ごとに変えられるようだし、懇親会も出来るバースペースもあってすごい。。。！

- サイボウズでは2013年くらいからGoを使っている。
- OSSとして2014年にはkintone Go SDK、2016年からはcybozu-go org(10 repo)としてもいろいろ公開している。
  - https://github.com/kintone/go-kintone
  - https://github.com/cybozu-go
- Kintone on AWSのYakumoは50%がGo。
- Monorepo構成。vendorを使わず全てのコードをgit repoに入れている。
- journaldとgoは相性が悪いので対策としてはjournald使わないほうがよい。
- Goがエラー時に`SIGPIPE`をシグナルしてもjornaldは`SIGPIPE`を`SuccessExitStatus`として取り扱っているので`Restart=on-failure`しないらしい。

サイボウズがこんなにもGoを使っているとは知らなかったのでお話を聞けて非常によかった。  
あまりメジャーではない（？）Monorepo構成をしていたり、積極的な技術チャレンジしていそうだった。

# Testing with microservices in merpay
株式会社メルカリ [kazegusuri](https://twitter.com/kazegusuri)

https://speakerdeck.com/kazegusuri/testing-with-microservices-in-merpay

メルカリさんはスカラーシップとして地方学生を無料招待されていた。  
社会人でもあまり行動圏外のイベントには参加できないので、こういう取り組みはすばらしいなあ

**Go Conference 2018 Spring スカラーシップのご案内**  
http://tech.mercari.com/entry/2018/03/19/164252

gRPC王と噂のkazegusuriさんの発表。  
自分もよく`mercari/go-grpc-interceptor`を見ている。  
https://github.com/mercari/go-grpc-interceptor

- メルカリでは2015年末頃にgRPC導入。
- Go + gRPC/k8s(GKE + EKS?)/Circle CI + Spinnaker/Stack Driver Logging, Profiler/Datadog/Pagerduty, Sentry
- メルペイで決済システムを作るうえでの難しさ
  - 一貫性とクリティカル。
  - 0.1%のエッジケースを考慮したシステム
  - マイクロサービスの中でトランザクションが分離していくなかでどうするのか。
- メルペイの決済システムは中核となる決済システムが各マイクロサービス間の処理の流れを状態遷移モデルで監視している。
- 共通テストフレームワークを作成し、状態遷移中の様々な異常状態を定義してテストしている。
- Failure Injection(MySQL)。MySQLの結果にエラーを注入する
  - https://github.com/shogo82148/go-sql-proxy
- Failure Injection(HTTP)はRoundTripper
  - https://golang.org/pkg/net/http/#RoundTripper
- gRPCへのFailure Injectionはインターセプターで
- テストの中でエラーをわざと発生させるが、ランダムでエラーを発生させるとテストが不安定になる。
- エラーの発生は決定的に行いたいので、`stacktrace(runtime.Callers)`で一度発生したパスでは二度と発生させない。
- 外部・内部サービスのテストはFakeをつかう
  - GCP Pub/SubはFakeサーバが提供されていたり。
  - https://github.com/GoogleCloudPlatform/google-cloud-go/blob/master/pubsub/pstest/examples_test.go#L28
- Fakeが本物と同じ動きをしてくれるのかもテストしている。本物とFakeに対して同じ処理を行って再現度が完璧なのか見ている。
- データベースを接続するようなテストではTestMainでschemaファイルを読みこんでテスト毎にDBを起動している。
- E2EテストではgRPCサーバを起動してクライアントからテストしている。
- Chaos Testでは`Shopify/toxiproxy`をつかって遅延させたり帯域制限したり落としたり。
  - https://github.com/Shopify/toxiproxy
- PostManでテストケースをGitHubで管理してQAテスト

具体的なOSSなどの紹介もあり、マイクロサービスにおけるテストについてものすごい情報量だった。  
そこまでテストできるのか！？というくらいのアプローチ、全部やるのか（やれるのか）は別として手札として取り込みたい。

# How to write Go code
[@kaneshin](https://twitter.com/kaneshin0120)  
https://speakerdeck.com/kaneshin/how-to-write-go-code

エウレカCTOのkaneshinさんの発表。  
発表のほぼ大半は実際にライブサーフィン？ライブブラウジング？で公式めぐりをされていた。  
Goはgodoc上でExampleがそのまま動かせたりするし、実コードにも補足コメントが色々書いてあって、
本当に公式だけで大半の調査ができるところが魅力だと思う。  
たしか発表中には出てこなかったと思うが、Wikiの[CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)も定期的に読んでおきたい。

- 愚者は経験に学び、賢者は歴史に学ぶ。
- どの記事が新しいのか、ちゃんとしているのか、鮮度などがよくわからない状態Goについて調べるならばやはり公式が一番。
  - https://golang.org/
  - https://godoc.org/
  - https://blog.golang.org/
  - https://github.com/golang/go/wiki
- godoc
  - [Effective Go](https://golang.org/doc/effective_go.html)
  - [リリースノート](https://golang.org/doc/devel/release.html)
- wiki
  - [Table Driven tests](https://github.com/golang/go/wiki/TableDrivenTests)
  - [LearnServerProgramming](https://github.com/golang/go/wiki/LearnServerProgramming)

余談？でthe Code Galaxiesというリポジトリの関連を可視化するサービスも紹介されていた。  
https://anvaka.github.io/pm/#/?_k=cj0p80


# GoらしいAPIを求める旅路
[@lestrrat](https://twitter.com/lestrrat)  
https://www.slideshare.net/lestrrat/goapi-go-conference-2018-spring?ref=https://gocon.connpass.com/event/82515/presentation/

[peco](https://github.com/peco/peco)や[Builders Con](https://builderscon.io/tokyo/2018)の牧さんの発表。

- GoらしいAPIとは
  - お題。 https://github.com/cenkalti/backoff
  - lestrrat-sanが書き直したもの https://github.com/lestrrat-go/backoff
- 本系backoffのAPIは`func Retry(f func() error, b BackOff) error`
- シグネチャ(引数や戻り値)合わせないといけないしそのために毎回クロージャをつくるのはGoっぽくない。。。
- その言語の自然な書き方が出来るようにAPIを作る
- Goでいうと必ずしも実装をひとつの構造体に隠蔽することが良いわけではない。

実際に再設計されたAPIは[lestrrat-go/backoff](https://github.com/lestrrat-go/backoff)のREADMEで見ることができる。  
本当に自然な感じのGoのコードになっている。
自分も設計を考えるときにとりあえずひとつのstructに入れてレシーバメソッド生やす、というような考え方をよくしてしまうので、とても参考になった。


# コードジェネレートとの付き合い方
[@pei0804](https://twitter.com/pei0804)  
https://www.slideshare.net/JumpeiChikamori/go-conference-2018-spring

goのコードからswagger定義を生成する`swaggo/swag`を開発されている方。  
[[swaggo]GoのGoDocを書いたら、Swaggerを出せるやばいやつ](https://qiita.com/pei0804/items/3a0b481d1e47e5a72078)

- コードを自動生成するOSSは多い。が、気になるときもある。
- Categoryを元にジェネレートした時、メソッド名はCategoriesになってほしいとか
- `swaggo/swag`を作っている。goaみたいなことをgoa以外でもswaggerドキュメントを作りたかった。
- ASTを作ってごりごりしている。必須と任意項目が混ざると大変。
- 構造体のパースをするときに何も意識していないと、相互参照体があったとき無限ループに…
- 実際にジェネレータを開発してみて感じたメリット
  - 機械的に生成できる効率化
  - 同じ書き方が保証できて品質担保
- 実際に開発してみて感じたデメリット
  - 結局パーサーなので、小さい変更でも実装コストがデカイ。生産性をペイできないときもありそう。
- 作るときは泥臭くがんばる
- 使う側の付き合い方としては自分もコミットするくらいの気持ちのが良い




# Gormの気持ちになってGormを使う
[@linyows](https://twitter.com/linyows)  
https://speakerdeck.com/linyows/become-a-gorm-feeling-and-use-gorm

- gormはgoで使うORMのライブラリ
  - http://gorm.io/
- goでは珍しくメソッドチェーンをする。
-メソッドチェーンようなのは他にはgomockとか？
- gorm使っているとメソッドが乱立する問題。
- goの言語思想(コンセプトの分離)とActive Recordパターンは相性がよくない
- Active Record pattern テーブル行をカプセル化したオブジェクトにドメインロジックのメソッドを生やすパターン
  - https://www.martinfowler.com/eaaCatalog/activeRecord.html
  - gormはレシーバが(`User`のような)テーブル行ではない
- 必要のないカプセル化をやめてDB操作はGormに任せることで処理をシンプルにする
- DBコネクションとデータ、ドメインロジックの分離をメリットと捉える
- いつも使うクエリはGORMのScopesに登録しておけば良い
  - http://doc.gorm.io/crud.html#scopes

クリーンアーキテクチャというかレイヤーを意識しようとしたらDB操作をAPI層に直に書きたくない気もするし、どうしたら良いのだろう？  
Tweetを見ても「`grom`…」というつぶやきが多かった気がする。

# Goroutine meets a signal 
[@codehex](https://twitter.com/codehex)  
https://speakerdeck.com/codehex/goroutine-meets-a-signal

Golet (https://github.com/Code-Hex/golet)  
[Go Conference 2018 Spring へ登壇しました](https://codehex.hateblo.jp/entry/2018/04/15/153000)  
[GoCon に参加して本当に良かった話](https://codehex.hateblo.jp/entry/2018/04/16/114616)

- 元ネタはPerlの[Proclet](https://metacpan.org/pod/Proclet)をGoに移植したgolet
  - https://github.com/Code-Hex/golet
- `SIGNAL`と`cancel()`を受け取る`context.Context`を内包した拡張構造体を定義した。
- `close`をつかうと`channel`を持っているgoroutineに全てに通知ができる。が。。。
- `context.Context`からキャンセルも受け取りたい。
- `SIGANL`をハンドルする`struct`に`context.Context`を含める
- `context.Context`をラップしているので、`context`としても機能する
- `SIGNAL`も受け取るので両方を`select`で待機できる
- ただ、`context`パッケージには`Do not store Contexts inside a struct type;`と書いてある…
  - https://golang.org/pkg/context/


ちょうど`context.Context`を構造体に含めちゃいけない理由を会社の人と話していた。  
その時は構造体のレシーバメソッドに呼び出し元から`context.Context`があったとき

- 構造体に内包した`context.Context`
- メソッドの引数で受け取った`context.Context`

どちらにどう従うのかが曖昧になってしまうから構造体に内包するのは良くないのでは、だと話になった。  
あと、構造体に含めるときは構造体オブジェクトと`context.Context`のライフサイクルが同じであることが前提なのも懸念事項にある？

**Why context.Context not in structs ?**  
https://gist.github.com/cia-rana/2ef3d6c33f93ce9add633989f9b8926e

**[Blog] Context should go away for Go 2**  
https://groups.google.com/forum/#!topic/golang-nuts/eEDlXAVW9vU

この辺読んだ気がする。曖昧な判断で構造体に内包するのはよくないのは確かそう。  
なお、発表のあと[牧さんと別のアプローチの話をされていたそう。](https://codehex.hateblo.jp/entry/2018/04/16/114616)

# Dgraph - A high performance graph database written in pure Go
[@munisystem](https://twitter.com/munisystem)  
https://speakerdeck.com/munisystem/dgraph-a-high-performance-graph-database-written-in-pure-go

DGraphというGo実装の分散グラフデータベースの紹介

https://github.com/dgraph-io/dgraph

- たくさんのWebアプリはグラフで表現できる関係をもっている
- が、グラフをRDBMSで表現するのはむずかしい。リレーションか新しい型が必要になるから。
- Wantedlly Peopleのようなユースケース(名刺を大量にWriteする)でパフォーマンスがでるような既存のサービスがない。
- そこでDGraphを採用した。
  - https://github.com/dgraph-io/dgraph
  - graphQLライクなクエリシンタックス
  - gRPCやHTTPもサポートしている。
  - Writeがハイパフォーマンス
  - 水平スケーリングも得意
- golangで書いてあるので読めばなんとかなるのではというところもDgraphの魅力。




# LT1: Whitespace言語処理系をGoで実装してみた
[snowcrush](https://twitter.com/snowcrush)
https://tanstaafl.0pt.jp/slides/whitespace.v2/#1

- Whitespace言語をGoで実装した話。
  - github.com/minoritea/whitespace
- Whitespace言語
  - https://ja.wikipedia.org/wiki/Whitespace
  - 空白、タブ、ラインフィードの三文字だけでプログラムを記述する
- いちおう実装は出来たがもろもろしんどかった
- Goで実装したのは初めてだと思っていたら結構先駆者がいた


swagもそうだけどパーサー作るの結構腕力が必要そう。だけど楽しそうだから自分も一度はやらないと。

# LT2: クローラー実装で学ぶgo
[川部 勝也](https://twitter.com/KKawabe108)  
https://www.slideshare.net/katsuyakawabe/study-golang-by-developing-mini-crawler-93887152

- クローラを実装するとは
  - 新しいプログラミング言語の勉強の題材にいい感じ
  - 新人や学生のベンチマークによさそう
- 教授からクローラ作成の課題をもらった。
- シングルページのクローリングの第一段階から再帰的なクローリングの並列化(平行か？)まで。
- 以下のことが勉強できた
  - Goを使ったソフトウェア設計
  - 文字列処理（URI周りで泥臭くなる）
  - 非同期処理(goroutine)
  - ファイル入出力
- 完成したクローラはこちら
  - https://github.com/Bo0km4n/avarus
  - 600行くらい、一週間で書けるはず。

クローラの実装ではhtnl、cssやjs内のURLの置換もしてローカルでページを再現していたらしいので、
正規表現やフロントエンド言語の構造解析の勉強になりそう。

# LT3: go-selfupdate-github でツールを「自己アップデート」できるようにする
[ドッグ](https://twitter.com/Linda_pp)  
https://speakerdeck.com/rhysd/go-selfupdate-github-de-turuwozi-ji-atupudetosuru

- goのCLIツールのアップデートはどうするのか？
  - `go get -u`
  - `brew`などのパッケージマネージャ
  - リリースページから直接ダウンロード
- どれも一長一短なので自動アップデートツールを作った
- リリースページから最新のバイナリを自動検出。
  - https://github.com/rhysd/go-github-selfupdate
  - CLIに組み込むことで以下を実現できる
  - GitHubのリリースページから最新のバイナリを自動検出
  - 自動ダウンロード
  - 実行ファイルの自動置き換え
  - ユーザーに確認して更新することも可能


GoのCLIツールに関わらず、自動更新機能がほしいときはあるのですごい嬉しいライブラリ！  
なお、`go get`に関して言うとブランチかtagに`go1`がつけて公開しておくとHEADへの追従は起きないらしい。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">前から結構言ってるんだけど、go getで最新取られたくない場合は go1 というタグを切れば良いです <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; Yoshifumi Yamaguchi (@ymotongpoo) <a href="https://twitter.com/ymotongpoo/status/985418278288764928?ref_src=twsrc%5Etfw">2018年4月15日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



# LT4: 学生版のGoConを開催したい！
[@Yamashou](https://twitter.com/Yamashou0314)  
https://speakerdeck.com/yamashou/xue-sheng-ban-goconwokai-cui-sitai

- Aizu.Goという活動をしている。学生とゲストエンジニアの発表。
  - https://twitter.com/hashtag/aizugo
  - (connpass的なページが見つからなかった)
- 会津限定じゃなくて全国の学生Gopherと交流してGo学生で盛り上がりたい
- StudentGoという学生向けのSlackチームもある
  - 現役Goエンジニアも学生と質疑応答、交流するために参加している
  - ぜひJOINを
  - https://docs.google.com/forms/d/e/1FAIpQLSdp6nGYN0sMgAQse7cV_YJKsdPpTzVoUasWXIldXSA0x72wdQ/viewform


# LT5: Goで始めるdockerとcontainerd
[@sakajunquality](https://twitter.com/sakajunquality)  
https://speakerdeck.com/sakajunquality/starting-docker-with-go

- Dockerというコンテナ技術
- 普通にDockerコンテナでgolangアプリ用のコンテナを作ると案外でかくなる
- multistage buildで`go build`を前処理として終わらせて、実行するコンテナとは別にする。
  - [Use multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/#use-multi-stage-builds)
- サンプルコードだと、380MBあったコンテナのサイズが11MBまで縮小した。

deeeetさんがおすすめしていたDockerfileの構成はこちらだった。

https://github.com/heptio/contour/blob/master/Dockerfile

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">heptio/contourのDockerfileがGoだ一番理想だと思う<a href="https://t.co/O1LemA30AZ">https://t.co/O1LemA30AZ</a> <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; Taichi Nakashima (@deeeet) <a href="https://twitter.com/deeeet/status/985421790372478976?ref_src=twsrc%5Etfw">2018年4月15日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

今自分で作っている開発用のコンテナは変更のたびに毎回`docker build`するのがめんどくさくて、
ソースコードマウントして`go run`で動かすように作っている。（コンテナリビルドしなくてもリスタートで変更が反映できる）  
本番用はちゃんとマルチステージで組まないと。


# LT6: Replaced Backlog Git Server from Perl to Go
[Yuichi Watanabe](https://twitter.com/vvatanabe_ja)  
https://slides.com/vvatanabe/replaced-backlogs-git-server-from-perl-to-go


- nulabのbacklogのgit serverをgolangにリプレースした話。
- Perl版とgo版を比較して
  - リクエスト処理は4倍くらい速くなった
  - CPU負荷は4割ほど削減
  - `git clone`、`git push`が2倍速くなった
  - アーキテクチャも変えているので一概には言えないが、golangいい感じ！

タイムオーバーだったので、最後のほうはスライドから読み取ったもの。ぜひ考察お聞きしたかった。

# 関連
- [Go Conference 2017 Autumn参加メモ #gocon](/2017/11/09/gocon2017-autumn/)

