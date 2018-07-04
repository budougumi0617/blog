+++
title = "mercari.go #1 参加メモ #mercarigo"
date = 2018-07-05T08:00:48+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["report","kubernetes", "go"]
tags = ["k8s","golang", "mercari", "gcp"]
+++




メルカリ内でGoがどのように使われているのかを聞くため[mercari.go #1](https://mercari.connpass.com/event/91306/)に参加してきた。  
詳しくは所感に書いたがいろいろな角度のGoの話を聞くことができてとても参加できてよかった。なお次回は8月を予定しているとのこと。

<!--more-->


<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">mercari.go 参加ありがとうございましたー！<br>8月くらいに第2回をやろうと思ってるのでお楽しみに！ <a href="https://twitter.com/hashtag/mercarigo?src=hash&amp;ref_src=twsrc%5Etfw">#mercarigo</a></p>&mdash; morikuni (@inukirom) <a href="https://twitter.com/inukirom/status/1014160235324325889?ref_src=twsrc%5Etfw">2018年7月3日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


|||
|---|---|
|URL|https://mercari.connpass.com/event/91306/|
|会場|株式会社メルカリ 東京都港区六本木6-10-1(六本木ヒルズ森タワー18F)|
|日時|2018/07/03(木) 19:30 〜 22:00|
|ハッシュタグ| [#mercarigo](https://twitter.com/hashtag/mercarigo)|
|Togetter|https://togetter.com/li/1243367|

# 所感
今回は3人の方から以下の内容を聞くことができた。  
フルスクラッチで書かれているらしいGoのAPIゲートウェイのアーキテクチャ、テストのための実装の詳細、
Go初心者から2年半Goの開発をして感じたことと、メルカリ内でのGoプロダクトについて異なるベクトルの話を3つも聞けて非常によかった。  
単純な実装に対する技術力だけではなく、プロダクトに対する課題解決の姿勢などにもレベルの高さを感じた。次も参加したい。  

- MAX約56,000req/secをさばく内製API Gatewayのアーキテクチャ思想と実装、旧アーキテクチャからの移行方法について
  - メルカリMicroservices PlatformテームTech LeadのdeeeetさんによるAPIゲートウェイの解説
  - なぜ自作したのか、実際にどうやって移行したのか、その背景まで聞くことができた
  - 如何に汎用性が高く拡張性に富んだAPI Gatewayにするか、けっこう踏み込んだ設計について聞くことができた
- GoでDBを扱うWebサーバを構成するときのテスト戦略について
  - Goでどのように状態をDBに依存しないテストを書くのか具体的なコードを例に知ることができた
  - やはり設計初期段階からしっかりとテストを想定したデータ構造、依存関係の整理しておくことが大事
  - 御本人は既知のアプローチと謙遜していたが、Functionalテスト(APIレベル)まで想定した実践的なServer Mockの書き方はあまりなかったのでは
  - `interface`定義しすぎるとモック書くのが辛くなるところをコードジェネレータ書いて解決するところに技術力を感じた
     - https://github.com/Code-Hex/funcy-mock
  - 自分でもあとでSeverMockできるようにコード書いてみよう
- Go未経験の状態から二年半メルカリアッテのサーバーサイドをGoで開発して学んだこと。
  - なぜGoがいいのか、エモい部分をしっかりと言語化されていた
  - たしかにドメインロジックに集中して書ける・（他人のコードを）読めるのがGoのいいところだと思った
  - 自分も「(自分の)Go言語(の実装力)がカバーできる問題領域」を広げていかないといけない
  - VimからIDE(Goland)に乗り換えているらしく自分もVimなのでちょっと考えた
    - VimでGoの補完などに不満をもったことはないけれど、リファクタリングをするときはIDE使いたくなってはいる


以下自分メモ


# API GatewayによるMicroservices化
[@deeeet](https://twitter.com/deeeet)

- https://go-talks.appspot.com/github.com/tcnksm/talks/2018/07/mercarigo/microservices-api-gateway.slide#1

メルカリMicroservices PlatformテームTechleadのdeeeetさんによるAPIゲートウェイの解説。
APIゲートウェイ使って実現したい世界やサービスメッシュなどについてはGCPUGで発表されていた資料などが詳しい。

- [Microservices on GKE at Mercari](https://speakerdeck.com/tcnksm/microservices-on-gke-at-mercari)
- [[k8s]GCPUG Tokyo Kubernetes Engine Day April 2018参加メモ #gcpug](/2018/04/28/gcpug-tokyo-kubernetes-engine-day-april-2018/)

## 構想
- 組織成長後も開発スピードを落とさないためにモノリスアーキテクチャからマイクロサービス化を図っている
- APIゲートウェイのマイクロサービスの中での役割
  - 単一エンドポイントでのRouting
  - 認証、ロギング・モニタリングなどのMiddleware(共通処理)を一括で行う
  - Proxy, Facadeパターンを適用したモノリスからの段階的なマイクロサービス化への補助
     - [Pattern: API Gateway / Backend for Front-End](http://microservices.io/patterns/apigateway.html)
- 最大約56,000リクエスト/秒をさばくハイパフォーマンスが必要
- 内製サービスとの連携、みんなが使える言語で拡張できることを考えGoで実装することに
- 基本構成は前段にGoogle Load Bralancer(GLB)。API Gatewayの後ろにGKE上の各サービス
- API Gateway自体もGKE上にいる

## 設計・実装
- Goでフルスクラッチ。Gopher friendlyであること
- 基本的に基本パッケージで全部やる。
- Core as a package
  - JP, US, merpayなどの個別でゲートウェイを作れるようにCore部分とそれを使った具体的な実装に分離している。
- SREがコアパッケージを実装し、Developerがそれを利用してサービスの実装を書く
- Excluding business logic
  - Coreパッケージを使った実装にはビジネスロジックは書かせないことで第二のモノリスになることを避ける
- Middleware driven
  - 認証・ロギングなどの共通処理はMiddlewareパターンで実装されている
     - [Writing middleware in #golang and how Go makes it so much fun.](https://medium.com/@matryer/writing-middleware-in-golang-and-how-go-makes-it-so-much-fun-4375c1246e81)
- Protocol transformation
  - API定義は基本的にProtocol buffer
  - API gatewayでHTTPを受けて中のサービス間はgRPCで通信している（構想当時はgRPCの事例も少なかったので）
  - Developerは以下を行うことでエンドポイントを追加できる
     - サービスのインターフェースをProtcol bufferで定義
     - エンドポイントの定義をAPI Gatewayの実装に追加
  - コアパッケージはエンドポイント定義を基に`http.Handler`を生成する
  - gRPCのエラーコードの変換をしたり、Request HeaderをもとにJSONのリクエストを受け付けている
- Request buffering
  - `vulcand/oxy`をforkしたrate limitなど標準pkgが提供していない便利機能を提供している
     - https://github.com/vulcand/oxy
  - GLBはリクエストバッファリングをしないのでAPI Gatewayでバッファリングをしている
  - メモリにリクエストを書き出して、溜まってきたらリクエストが溜まったらリクエストをバックエンドを送る
- API GatewayへのMigration
  - APIのドメインはAWSのRoute53で管理されているので、Weighted Recordsで流量制御しながら1ヶ月かけて移行していった。
     - https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-values-weighted.html#rrsets-values-weighted-weight
- パフォーマンス
  - Datadogによる可視化、Datadog APMの分散トレーシングは行っている
  - ProfilingはStackdriver profilierも使っている
     - https://cloud.google.com/profiler/
  - CPUレベルでどのファンクションにどれくらい時間が書かているか可視化できる。
  - パフォーマンスにもそこまで影響していない
  - Googleなどと同様に本番GatewayでProfilerを動かしている。
- 質疑応答
  - protoファイルどう定義している？どうサービス間で互換性を保っている？
     - protoファイルは一つのリポジトリで全てのprotoファイルを管理してCIでクライアントコードなどが各リポジトリに配布される
  - gRPCでデカいコールのチャンクどうしている？
     - 例えば画像などを含んだデカいリクエストのチャンクはまだ実装していない
  - フォームやWebHookなどでURLパラメータにリクエスト内容が含まれているときどうgRPC用のリクエスト定義にマッピングしている？
     - `metadata`経由でマッピングする予定


# もう一度テストパターンを整理しよう
[@codehex](https://twitter.com/codehex)

- https://speakerdeck.com/codehex/mou-du-tesutopatanwozheng-li-siyou-webappbian  
- [mercari.go #1 で「もう一度テストパターンを整理しよう」というタイトルで登壇しました](https://codehex.hateblo.jp/entry/2018/07/03/211839)

GoにおけるWebAppのテストの実装パターンとテスタブルな設計、テストに使うmock実装について。（コードベースの話だったのでスライド中のコードを見たほうがよい）

- Serviceに`*sql.DB`なんかを持たせているとテストで死ぬ
- `sql.DB`、`sql.Tx`をインターフェースによって抽象化することで実装を分離する
- このときinterfaceは単一責務の複数のinterfaceに分けて抽象化すること
  - Goのinterfaceは一つのinterfaceに複数の責務を持たせないのがベストプラクティス
- interfaceを引数に持つラッパーメソッドを作成し、Serviceオブジェクトにinterfaceのフィールド(具象オブジェクト)をもたせることで具象オブジェクトとロジックの密結合を防ぐことが出来る([P30(https://speakerdeck.com/codehex/mou-du-tesutopatanwozheng-li-siyou-webappbian?slide=30)])
- `sql.Tx`を返すinterfaceを定義しておくとトランザクションを使うときもモック出来る。
- ServerのFunctional Test(APIテスト。Req/Respのテスト)はエンドポイントごとにTable Drivenつくっていく
- 各Serviceインターフェースを満たす`ServerMock`を作成する。このときServiceの各メソッドの挙動を注入できるように設計しておくこと([P68](https://speakerdeck.com/codehex/mou-du-tesutopatanwozheng-li-siyou-webappbian?slide=68))
  - テストケースごとにSeverの挙動を制御できる
- 擬似的に`context`などを発行するUtilsがあると便利
- テストケースの実行順はシャッフルすることで実行順依存の結果を排除する
- ServiceごとにMock定義を書くのはめんどくさいのでジェネレータも作った
  - https://github.com/Code-Hex/funcy-mock

# Go言語による2年半の新規プロダクト開発とその総括
[@cowsys](https://twitter.com/cowsys)

https://speakerdeck.com/cowsys/goyan-yu-niyoru2nian-ban-falsexin-gui-hurotakutokai-fa-tosofalsezong-gua

株式会社ソウゾウ時代に二年半メルカリアッテでGoの開発をして学んだこと。

- メルカリアッテ二年半で2年半バックエンドエンジニアとしてGoを書いた
  - それまではPHPerでGo未経験
- 4つのポイントでGoに燃えた
- simpleな言語仕様/思想で複雑なドメインを解決する
  - 言語仕様が薄く、頭に入れておくべき情報がコンパクト
  - interfaceや構造体埋め込みを利用した設計をしていれば、頭に入れておくべき情報もコンパクトで済む
  - ”モダンな実装方式を理解”するのも薄くすんだので、ドメインや周辺アーキテクチャの理解に時間を当てられた
  - 「愚直に書く」という積み重ねを着実にしていけばより複雑な問題も解決出来るようになる
     - 実装方式が陳腐がしないため
- コンピュータリソース・Go言語のツールキットを最大限活用した、実装能力のempowerment
  - コンパイルやGoの各種ツール、IDEを活用して自分のアタマを本質的な問題解決にフォーカスしてアウトプットを最大化したい
  - Golandを使って、「間違い」の混入、「思い出す」ための消費をより少なく出来ないか試している
- High Perfomanceなプログラミング
  - Dave Cheney-sanのトレーニングを受けて目覚めた。
     - [High Performance Go](https://go-talks.appspot.com/github.com/davecheney/high-performance-go-workshop/high-performance-go-workshop.slide#1)
  - ハードウェアの性能を最大限引き出すためのプログラミング
  - "札束"的な解決に頼らずパフォーマンスを出すためのプログラミング
- チーム開発におけるGo言語
  - 取り巻く・利用する情報が多いほどチーム内でブレのない認識統一が出来るのは強力
     - 型（コンパイル）による裏付け
     - in/outが自明であること
  - 消耗しなかった集中力/節約できた時間でよりプロダクト/teamとしてアウトプットが出せると感じた
- 質疑応答
  - 逆にGoで不満があるところは？
     - さすがに(`html/template`pkgを使って)Web ViewをGoで書くのは辛みを感じる


