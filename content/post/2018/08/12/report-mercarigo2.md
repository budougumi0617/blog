+++
title = "mercari.go #2 参加メモ #mercarigo"
date = 2018-08-12T12:04:41+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["report", "go"]
tags = ["golang"]
+++

mercari.go #2に参加してきた。今回はGoの話だけでなく、GraphQLについても勉強することができた。  
次回は9月開催にGopherCon2018の参加報告を予定しているとのこと。
<!--more-->

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">参加ありがとうございました！次回は9月にGopherCon 2018の報告会を予定しているのでお楽しみに！ <a href="https://twitter.com/hashtag/mercarigo?src=hash&amp;ref_src=twsrc%5Etfw">#mercarigo</a></p>&mdash; morikuni (@inukirom) <a href="https://twitter.com/inukirom/status/1027938248989921280?ref_src=twsrc%5Etfw">2018年8月10日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


|イベント名|mercari.go #2|
|---|---|
|URL|https://mercari.connpass.com/event/95665/|
|会場|株式会社メルカリ 東京都港区六本木6-10-1(六本木ヒルズ森タワー18F)|
|日時|2018/08/10(金) 19:30 〜 22:00|
|ハッシュタグ|[#mercarigo](https://twitter.com/hashtag/mercarigo)|
|Tweetまとめ|[mercari.go #2 ツイートまとめ](https://twitter.com/i/moments/1027880492157231105)|

# 所感
今回はQAやインフラよりの部分でGoがどのように使われているか、使うことができるかの紹介が多かった。  
サービス自体のコードだけでなく、開発の効率化にもしっかりと開発工数をあけて問題解決に取り組んでいるのを聞くことができてさすがだなという感じだった。  
インフラまわりのミドルウェアやツールでもGoの1バイナリになる成果物、高い平行性などの特徴を活かして課題解決を行なっているということだった。  
開発中にある日常的な悩みをGoでどんどん解決できているのは羨ましい（と言っていないで自分もやらないといけない）。  
また、GraphQLの発表もあり、実装で考慮すべき点など非常にわかりやすかった。  
GraphQLを遊ぶには[GitHub API v4 API](https://developer.github.com/v4/explorer/)のExplorerがインタラクティブな補完・仕様の参照ができて良いらしい。これはGraphQLに限らずAPI仕様を公開するときはこのようなUXを提供できるとみんなに興味を持ってもらえるんだろうなと思った。  
[How to GraphQL](https://www.howtographql.com/)は会社の先輩もわかりやすいと言っていたので機会があったら触ってみたいと思う。

以下、メモ


# メルカリにおける開発環境/QA環境と、そこで使われるGoのツールについて
[@masudak](https://twitter.com/masudak)

## Software Engineer in Test(SET)とは
  - テストが作りやすい環境を作る
  - 他社ではSDET, SWET, SETIなどの呼び方をしている

## SET設立の背景
- SETチームの設立背景と次世代のSETに向けて
  - https://tech.mercari.com/entry/2018/06/14/190000
- UK向けサービスを作ることになった
- 当時QAは４人日くらいでマニュアルテストをしていた
- このままの状態でサービスのリージョンが増えるとスケールしないので効率化が必要だった

## 当時の開発フローと開発環境
- PM(プロダクトマネージャー？)が要求出す
- エンジニア/デザイナーがローカルに開発環境を構築して開発
- QAエンジニアがマニュアルでテストしている
- PMが確認してリリース
- ローカル開発環境
  - Jenkinsで作ったDockerコンテナをレジストリにおいてある
  - `make init, make start`するとローカルでdocker composeが立つ。IPもふられる
- QAチームのテスト
  - Web UIでトピックブランチを指定すると自動的にQA環境をGCP上に作ってくれる

ここまでの話はPHPのモノリシックなサービス開発環境向けの開発フローと環境

## マイクロサービス開発初期の開発環境
mercariは2017年からマイクロサービス開発に移行を始めている。
マイクロサービス開発では以下の仕組みを使って開発環境を構築していた。

- PRのCircleCIでビルドした結果を用いてKubernetes(k8s)上のserviceとdeploymentを作成
- k8sはもろもろの設定をPR名で置換できるテンプレートを持っている
- デプロイされたPR別の開発中の環境にはリバースプロキシで接続できる
  - クライアントのHTTPヘッダーで指定されたPR番号に接続できるようになっている
  - Goならばリバースプロキシ簡単に作れた。

ただし、この仕組では利用が終わったPR用の開発環境を消す仕組みがちゃんとなかった。

## 現在のマイクロサービス開発の開発環境
- PR replication controllerを自作した（k8sのカスタムコントローラを作った
- Kubernetes Controller for Pull Request Based Environment
  - https://www.slideshare.net/VishalBanthia1/kubernetes-controller-for-pull-request-based-environment
- 事前にリポジトリをアノテーションとしてReplicaSetにつけておく
- 対象のリポジトリのPRを定期的に監視し、OpenなPRがあったサービスを立ち上げる
  - リポジトリのPRの状態をDesiredとして、k8s上にデプロイメントを展開する
  - これなら閉じたPRがあればそのままk8s上のデプロイメントもなくなる
- OSS化するかもしれない
- grpc-translator
  - gRPCだとPostmanとかで簡単にデータを入れられない
  - gRPC-HTTP変換ツールを作った
  - GRPC server reflectionでいい感じにHTTPとgRPCを接続することができる
      - https://github.com/grpc/grpc/blob/master/doc/server-reflection.md

PRから開発環境をつくる詳細はcrash.academyさんの動画でも聞くことができる

- GCPUG Tokyo June 2018 How to utilize GKE for QA environment
  - https://crash.academy/class/287


# GoでGraphQLサーバを立てるぞ！
[@vvakame](https://twitter.com/vvakame)

- GoでGraphQLサーバを立てるぞ！ / Building GraphQL server by go
  - https://speakerdeck.com/vvakame/building-graphql-server-by-go

## GraphQLって何？
- https://graphql.org/
- Facebookが2015年に公開した仕様
- グラフ構造に対するQuery Language
  - 深さ2で友達の友達のデータがほしいとか
- RESTだと複数回コールが必要でもGraphQLなら一回でとれる
- とは言え
  - N+1対策とかちゃんとやらないと
    - APIコールが一回でも内部的なDBアクセスが最適化されるわけではない
  - RESTの上位互換ではない
- メルカリでは一部プロダクションでも使っている

## Clientの流れ
- Schema定義とGrahQLサーバがある
- サーバに対してオペレーションを投げる
- Query, Mutation, Subcribeなどの3種類のコールがある
  - Queries and Mutations
      - https://graphql.org/learn/queries/
  - Adding Subscriptions To Schema
      - https://www.apollographql.com/docs/graphql-subscriptions/subscriptions-to-schema.html
- Queryによって得られるデータ構造が変わるので、言語によっては使いづらい可能性。
- 返り値の形にぴったりの型がほしい

## サーバ側の原理
- 受け取ったクエリに対してTree構造を返す
- サーバは大小様々なResolverを組み合わせてレスポンスのJSONを作る
  - Root fields & resolvers
      - https://graphql.org/learn/execution/#root-fields-resolvers
- Resolverの集合体がサーバの実装となる

## GraphQLのどこがよい？
- 単一エンドポイント
- Introspection
  - スキーマを知るにもGraphQLが使える
  - https://graphql.org/learn/introspection/
- 強い型付け


## GitHub v4 APIを触ってみる
- Exploerページがある
  - https://developer.github.com/v4/explorer/
  - 仕様とエディタが一緒になっているので遊びやすい
  - インタラクティブな補完のドキュメント
- クエリを投げるだけならばGoの標準パッケージだけでできる
  - https://gist.github.com/vvakame/aaaa632bae97ee06c701f339fbae4dbc
- なぜGoでGraphQLを扱うかというと、GoならばResolverを並列で動かせそう
- いい感じだと思っているGoのライブラリはgqlgen。コミッターにもなった
  - https://github.com/99designs/gqlgen
- コード生成+手書きを程よく
  - 既存のstructを使いまわしたりすることができる
- 興味が出たらこれやってみるといいかも
  - https://gqlgen.com/getting-started/

## ベストプラクティス
- Relayを採用しなくても勉強したほうがよい
  - https://facebook.github.io/relay/
  - Reactのライブラリ
- GraphQLをクライアントがどう使うかわかっていないと意味がわからない仕様がわかるから
- Relay本体仕様以外にも以下を見ておくと良し
  - https://facebook.github.io/relay/docs/en/graphql-server-specification.html#further-reading
  - Relay cursor connection
      - https://facebook.github.io/relay/graphql/connections.htm
  - Relay global object identification
      - https://facebook.github.io/relay/graphql/objectidentification.htm
  - Relay input object mutation
      - https://facebook.github.io/relay/graphql/mutations.htm
- gqlgenはResolverを並列処理するので、DBアクセスを集約するのが難しい
- REST APIとの対比はどうかというと
  - エラーに対する考え方が根本的に違う
  - 正しいRequestの投げ方という概念はあまりない。驚き最小の原則とデバッガビリティ
- Schema設計は
  - Schemaから読み取れる情報を増やす
  - 一つのテーブルを１タイプにしたほうが良さそう
- テストは
  - モックとかテストはまだよくわからない
- 実装は
  - 直実装のほうがBFFよりラクそうだけど
  - マイクロサービスにするとGraphQLサーバが結局BFF的になりそう
- おすすめ
  - https://www.howtographql.com/

# Software Engineer, Infrastructure
[@cubicdaiya](https://twitter.com/cubicdaiya)

## Software Engineer, Infrstructureとは
- ソフトウェアエンジニアリングによってインフラの問題解決やミドルウェアの開発
- SREとの違いは運用よりも開発の比重が多いロール
- 今は3人(一人はSREと兼任)
- Go言語が中心
- ネットワークやシステムプロウグラミング寄りの知識やスキルが必要な感じ
- パフォーマンスチューニングとかボトルネックの解決とか（をソフトウェアエンジニアリングで）

## mercariとGo
- 最近メルカリはPHPなモノリシックなサービスをGoのマイクロサービスにしているが。。。
- 2014年からすでにGoを使っている
- 裏側のツールでGo(やNode.js)を使い始めていた
- 1000万超えるプッシュ通知の高速実現とか
- パフォーマンスやスケーラビリティからみたPHPとは
  - PHPも7になってだいぶ速くなった
  - ただ単体性能が向上しただけでシングルスレッドなどはそのまま。ウィークポイント
- パフォーマンスやスケーラビリティからみたGo
  - 数千req/secさばいても全然CPU使用率などが余裕
  - レスポンスタイムもavgで4ms
  - 非同期や平行性の高さを実現できる
- ツール開発言語としてのGo
  - １バイナリで楽
  - RPMで配布している。
- DockerとMakeによるRPMパッケージのビルドシステム
  - makeすると簡単に複数のディストリビューション向けにビルドできるようにしてある
  - 複数のOS向けのRPMをMacOS上でもできる
- 実際にyum installできるようになるまでの流れ
  - Travis CIでビルドする
  - ビルド結果のオブジェクトがS3にアップロードされる
  - それが各リージョンのyum repoが配布される

## メルカリの裏側で動いているGoによるツールやミドルウェアの紹介

### slackboard
- https://github.com/cubicdaiya/slackboard
- Incoming Webhoookプロキシ兼クライアント
- Slackへの通知処理や必要な情報を一元管理化
- クライアントプログラムを統一することでメンテナビリティも向上した
- TOMLでURLとか設定することができる
- バッチとかcronjobで終了コードが1のときにSlackに通知するときなどに利用している
- 各リージョンごとにGCPのインスタンスを立てて運用している（GLBがいい感じにリージョンが近いboardを選んでくれる
- （実際に使われてるクライアントツールはperl)

### pvpool
- pvpool〜メルカリの商品閲覧数カウントアップの裏側〜
  - https://tech.mercari.com/entry/2018/02/26/110237
- 商品閲覧数をカウントアップする仕組み
- 商品閲覧数のカウントアップのワークロードと技術的な要件
  - 商品閲覧に関するリクエストは全APIのTOP3に入る
  - 数秒以内にリアルタイム閲覧数を表示したい

###  go-htttpstat
- https://github.com/tcnksm/go-httpstat
- 小規模なHTTPサーバ向けのメトリクス取得ライブリ
- ちょうどいい感じのなかったので自作し
- リクエストカウントとかステータスカウント、速度とか
- percentileの計算はそこそこ重たい
- 最終的にはmackerelに流してグラフ化している

### Mackerelプラグイン
- mackerelプラグインもGoで開発
- 公式ヘルパーがあるのでそれを使う

### mfc
- fastly使っている
- 毎回curl叩くのめんどくさくなってきたのでCLIを作った
- 欲しくなった機能追加するようなゆるふわで作ってる
- API KEYを毎回curlに含めるのはめんどくさいのでiniファイル読んだりとか
- 機能領域ごとにサブコマンドを定義してる
- `aws`コマンドとか`gcloud`コマンドみたいなイメージ

### imageflux-cli
- https://github.com/mercari/imageflux-cli
- 画像変換サービスのCLIツール

### その他のGo製ツール
- gaurun
  - https://github.com/mercari/gaurun
  - ハイパフォーマンスGaurun〜メルカリの大規模プッシュ配信を支えるミドルウェア〜
      - https://tech.mercari.com/entry/2016/11/08/170343
- chocon
  - https://github.com/kazeburo/chocon
  - 【資料公開します】AWS Dev Day Tokyo 2017 にて登壇しました／choconの簡単なご紹介
      - https://tech.mercari.com/entry/2017/06/05/110000
- API Gateway
  -  API GatewayによるMicroservices化
      - https://go-talks.appspot.com/github.com/tcnksm/talks/2018/07/mercarigo/microservices-api-gateway.slide#1

# その他
- 次回のmercari.goは9月を予定していて、GopherCon 2018の参加報告になる予定とのこと
- また、10月にはmercari Tech Confも開催予定とのこと
  - https://techconf.mercari.com/2018/

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">参加ありがとうございました！次回は9月にGopherCon 2018の報告会を予定しているのでお楽しみに！ <a href="https://twitter.com/hashtag/mercarigo?src=hash&amp;ref_src=twsrc%5Etfw">#mercarigo</a></p>&mdash; morikuni (@inukirom) <a href="https://twitter.com/inukirom/status/1027938248989921280?ref_src=twsrc%5Etfw">2018年8月10日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 関連
- [mercari.go #1 参加メモ #mercarigo](/2018/07/05/report-mercarigo-1/)
- [[Go][gRPC]Server Reflection Tutorialをやってみた](/2018/01/04/server-reflection-tutorial/)
