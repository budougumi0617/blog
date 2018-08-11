+++
title = "mercari.go #2 参加メモ #mercarigo"
date = 2018-08-12T12:04:41+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["report", "golang"]
tags = ["go"]
+++

<!--more-->

|||
|---|---|
|イベント名|mercari.go #2|
|URL|https://mercari.connpass.com/event/95665/|
|会場|株式会社メルカリ 東京都港区六本木6-10-1(六本木ヒルズ森タワー18F)|
|日時|2018/08/10(金) 19:30 〜 22:00|
|ハッシュタグ|[#mercarigo](https://twitter.com/hashtag/mercarigo)|

# 所感

# メルカリにおける開発環境/QA環境と、そこで使われるGoのツールについて
[@masudak](https://twitter.com/masudak)

- Software Engineer in Test(SET)
  - テストが作りやすい環境を作る
  - 他社ではSDET, SWET, SETIなどの呼び方をしている
- SETチームの設立背景と次世代のSETに向けて
  - https://tech.mercari.com/entry/2018/06/14/190000
- 背景
- UK向けアプリを作ることになった
- QAは４人日くらいでマニュアルテストをしていた
- 自動化などいろいろすることになった
- 開発フロー
- PMが要求出す
- エンジニア/デザイナーが簡単にローカルの開発環境を作れるようにした
- QAエンジニアがマニュアルでテストしている
- ローカル開発環境
- Jenkinsで作ったDockerコンテナをレジストリにおいてある
- `make init, make start`するとローカルでdocker composeが立つ。IPもふられる
- QA
- Web UIでトピックブランチを指定すると自動的にQA環境をGCP上に作ってくれる
- ここまではPHPのモノリシックな環境向けの開発フローと環境
- 2017年から
- PRのCircleCIでビルドした結果をk8s上のserviceとdeploymentを作成
- リバースプロキシで接続できる
- PR名で置換できるテンプレートを持っている
- クライアントのHTTPヘッダーで指定されたPR番号に接続できる用になっている
- Goならばリバースプロキシ簡単に作れた
- 消す仕組みがちゃんとなかった。
- 二弾
- PR replication controllerを自作した（k8sのカスタムコントローラを作った
- https://www.slideshare.net/VishalBanthia1/kubernetes-controller-for-pull-request-based-environment
- 事前にリポジトリをアノテーションをレプリカセットにつけておく
- 対象のリポジトリのPRをみてそのPRの状況をみてサービスを立ち上げる
- OSS化するかも
- grpc-translator
- gRPCだとPostmanとかで簡単にデータを入れられない
- gRPC-HTTP変換ツールを作った
- GRPC server reflectionでいい感じにHTTPとgRPCを接続することができる



PRからいろいろ作る動画
https://crash.academy/class/287


# GoでGraphQLサーバを立てるぞ！
[@vvakame](https://twitter.com/vvakame)

- https://speakerdeck.com/vvakame/building-graphql-server-by-go

## GraphQLって何？
- Facebookが2015年に公開した仕様
- グラフ構造に対するQuery Languege
- 深さ2で友達の友達のデータがほしいとか
- RESTだと複数回コールが必要でもGraphQLなら一回でとれる
- とは言え
- N+1対策とかちゃんとやらないと
- RESTの上位互換ではない
- メルカリでは使っている？
- 一部プロダクションでも使っている
- Clientの流れ
- Schema(+サーバ）がある
- それにオペレーションを投げる
- Query, Mutation, Subcribeなどの3種類のコールがある
- Queryによって得られるデータが変わる
- クエリ個別に返り値のデータの型がほしい
  - 言語によっては使いづらい可能性。返り値の形にぴったりの型がほしい
- サーバの原理
- クエリに対してTree構造を返す
- 大小様々なResolverを組み合わせてレスポンスのJSONを作る
- Resolverの集合体がサーバの実装となる
- どこがいいの？
- 単一エンドポイント
- Introspection。
  - スキーマを知るにもGraphQLが使える
  - https://graphql.org/learn/introspection/
- 強い型付け




## GitHub v4 APIを触ってみる
- Exploerページがある
  - 仕様とエディタが一緒になっているので遊びやすい
  - インタラクティブな補完のドキュメント
- クエリを投げるだけならばGoの標準パッケージだけでできる
- https://github.com/99designs/gqlgen
- GoならばResolverを並列で動かせそう
- ボイラーテンプレートは嫌
- 完全コード生成も嫌
  - 既存のstructを使いまわしたりすることができる
- コード生成+手書きを程よく
- これやってみるといいかも
- https://gqlgen.com/getting-started/
- ベストプラクティス
- Relayを採用しなくても勉強したほうがよい
- Reactのライブラリ
- クライアントがどう使うかわかると意味がわからない仕様がわかる
- gqlgenはResolverを並列処理するので、DBアクセスを集約するのが難しい
- REST APIとの対比はどうかというと
- エラーに対する考え方が根本的に違う
- 正しいRequestの投げ方という概念はあまりない。驚き最小の原則とデバッガビリティ
- Schema設計は
- Schemaから読み取れる情報を増やす
- 一つのテーブルを１タイプにしたほうが良さそう
- テストは。。。
- モックとかテストはまだよくわからない
- 実装は
- 直実装のほうがBFFよりラクそうだけど
- マイクロサービスにするとGraphQLサーバが結局BFF的になりそう
- おすすめ
  - https://www.howtographql.com/
-
# Software Engineer, Infrastructure
[@cubicdaiya](https://twitter.com/cubicdaiya)

- Software Engineer, Infrstructure
- ソフトウェアエンジニアリングによってインフラの問題解決やミドルウェアの開発
- SREとの違いは運用よりも開発の比重が多いロール
- 今は3人(一人はSREと兼任)
- Go言語が中心
- ネットワークやシステムプロウグラミング寄りの知識やスキルが必要な感じ
- パフォーマンスチューニングとかボトルネックの解決とか（をソフトウェアで）
- 最近メルカリはPHPなモノリシックなサービスをマイクロサービスにしているが。。。
- 2014年からGoを使っている
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
  - makeすると簡単にできる。複数のOS向けのRPMをMacOS上でもできる
- 実際にyum installできるようになるまでの流れはTravis CIでビルドしてS3にアップロードされる
- それが各リージョンのyum repoが配布される

## メルカリの裏側で動いているGoによるツールやミドルウェアの紹介
- slackboard
- https://github.com/cubicdaiya/slackboard
- incoming Webhoookプロキシ兼クライアント
- Slackへの通知処理や必要な情報を一元管理化
- クライアントプログラムを統一することでメンテナビリティも向上した
- TOMLでURLとか設定することができる
- バッチとかcronjobで終了コードが1のときにSlackに通知するとか
- 各リージョンごとにGCPのインスタンスを立てて運用している（GLBがいい感じにリージョンが近いboardを選んでくれる
- （実際に使われてるクライアントツールはperl)
- pvpool
- 商品閲覧数をカウントアップする仕組み
- 商品閲覧数のカウントアップのワークロードと技術的な要件
- 商品閲覧に関するリクエストは全APIのTOP3に入る
- 数秒以内にリアルタイム閲覧数を表示したい
- https://tech.mercari.com/entry/2018/02/26/110237
- go-htttpstats
- 小規模なHTTPサーバ向けのメトリクス取得ライブリ
- ちょうどいい感じのなかったので自作し
- リクエストカウントとかステータスカウント、速度とか
- percentileの計算はそこそこ重たい
- 最終的にはmackerelに流してグラフ化している
- mackerelプラグインもGoで開発
- 公式ヘルパーがあるのでそれを使う
- mfc
- fastly使っている
- 毎回curl叩くのめんどくさくなってきたのでCLIを作った
- 欲しくなった機能追加するようなゆるふわで作ってる
- API KEYを毎回curlに含めるのはめんどくさいのでiniファイル読んだりとか
- 機能領域ごとにサブコマンドを定義してる
- `aws`コマンドとか`gcloud`コマンドみたいなイメージ
- imageflux-cli
- 画像変換サービスのCLIツール
- https://github.com/mercari/imageflux-cli
- gaurun
- chocon
- API Gateway

09/21 GopherCon 2018報告会
10/04 https://techconf.mercari.com/2018/



