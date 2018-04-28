+++
title= "[k8s]GCPUG Tokyo Kubernetes Engine Day April 2018参加メモ #gcpug"
date= 2018-04-28T15:52:39+09:00
draft = false
slug = ""
categories = ["gcpug", "report","kubernetes", "gcp"]
tags = ["k8s","spinnaker", "gcp", "gke"]
author = "budougumi0617"
+++

GCPUGに行ってMercariのマイクロサービス環境、有識者方のGKEパネルディスカッションを聞いてきたので参加メモ。

|||
|---|---|
|URL|https://gcpug-tokyo.connpass.com/event/81224/|
|会場|六本木ヒルズ 森タワー 18F メルカリ|
|日時|2018/04/26(木)19:00 〜 22:00|
|資料|[Microservices on GKE at Mercari](https://speakerdeck.com/tcnksm/microservices-on-gke-at-mercari)|
|ハッシュタグ|[#gcpug](https://twitter.com/hashtag/gcpug)|

# Microservices On GKE At Mercari
[@deeet](https://twitter.com/deeeet)  
https://speakerdeck.com/tcnksm/microservices-on-gke-at-mercari

## なぜMicroserviceを目指すのか。
- 最初はモノリスなシステムでよい。最初はむしろそのほうがラク。
- ビジネス、機能、組織の成長、そのなかでモノリスのシステムが肥大化していく
- 膨張したモノリスシステムでは影響範囲が見えなくなってくる。
- 複雑化したシステムでは機能追加、修正、テストや新メンバーのオンボーディングが難しい
- コードのオーナーシップの低下、コミュニケーションのオーバーヘッドも発生してくる
- これらの問題を解決するためにマイクロサービス化していく

## マイクロサービス
- マイクロサービスはアプリケーションの構成を疎結合かつ各々の境界を定義した小さなサービスの集合体として構成する。
- 個別の小サービスなのでテスト、アイソレーション、デプロイもらく、

### マイクロサービスへの移行
- Gatewayパターン、Stranglerパターンを利用することで少しずつ移管していく。
- [API Gateway Pattern](http://microservices.io/patterns/apigateway.html)
  - システムへのアクセスを全てAPIゲートウェイ経由にする
  - システムの利用者側はモノリスからなのか切り出されたサービスからなのか意識せずにシステムにアクセスできる
- [Strangler Pattern](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/strangler)
  - あるサービス、コンポーネントにファサード(プロキシ）をかぶせる
  - ファサードで機能の実現、あるいは別のサービスの機能を利用するようにする
  - 利用者側はファサードから先がどのように置き換わっているか意識することはない

## メルカリのマイクロサービス構成
- 既存のモノリスサービスはさくらインターネットの専用DCを使っている
- 新しく作ったAPIゲートウェイやマイクロサービスは全てGCP上にある。
- APIゲートウェイはフルスクラッチのGo製。ゲートウェイ自体に必要な認証機能なども別サービスとして分割管理されている。
- サービスメッシュ(後述)を使うつもりなので、なるべくツールへの依存を避けてピュアGoでゲートウェイしたかった。
- データは基本的にマネージドのGCPのサービスを利用
- マイクロサービスは基本的にコンテナで構築される
- 各言語ごとのデプロイの仕方をつくるとかめんどくさい。
- インフラからみて「コンテナをデプロイする」という抽象レベルで各サービスのオペレーションが出来る

## gRPC(Protocoll Buffers)
- https://grpc.io/
- データをシリアライズして高速なRPCを実現するプロトコル。
- Protocoll BuffersでAPI仕様の策定・各言語に対してAPIコードの自動生成ができる
- APIゲートウェイと各マイクロサービスはGKE上に展開され、gRPCで通信している。
- gRPCを利用することでREST疲れやJSONを使っていたときのバリデーション(`1`?`"1"`?)の心配は不要になる

## Spinnaker
- https://www.spinnaker.io/reference/providers/kubernetes-v2/
- 継続的デリバリーツール。GUIでパイプラインの可視化、K8sクラスター上へカナリーリリースなどができる。
- デプロイはCircleCIでビルドしたコンテナをSpinnakerでデプロイしている
- [k8s Provider V2](https://www.spinnaker.io/reference/providers/kubernetes-v2/)を使うとYAMLでデプロイ設定の指定も可能
- Kayentaを使えばカナリリースをスコア化し自動カナリーリリースも出来る
  - [Automated Canary Analysis at Netflix with Kayenta](https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69)

## Istio(サービスメッシュ)
- https://istio.io/
- マイクロサービス間を仲介、もろもろの処理をしてくれるサービスメッシュを実現するOSS
- Istioを使えば、サービス間のリクエストをサイドカーパターンで注入したEnvoyで制御してくれる
- 認証とか、[サーキットブレーカー](http://microservices.io/patterns/reliability/circuit-breaker.html)とか。

## GKE
- 基本的に一カ国でひとつのプロダクションクラスター。
- 日本は開発用クラスターはあるけど、今後はProductionクラスターひとつにまとめたい。
- クラスターが増えるたびにLet's encryptとかしたくないし。
- ノードプール戦略。基本はn1-standard。MLだけHighMem
- 各サービス（各チーム）ごとにNamespaceを作っている
- それぞれのネームスペースにRBACつけていきたい（まだ）
- GCPのIAMはだいぶざっくりしているので、サービスごとにGCPプロジェクトを作りIAMアサイン。

## Terraform
- https://www.terraform.io/
- マネージドクラウドも含めてInfrastructure as Codeが出来る
- Cloud SQL作ったりSpannerのインスタンスつくるようなのは全部Teraformで行っている
- 連用専用のRepoで管理。PR作ると自動でCIがTeraformの構成に則ってオペレーションしてくれる
- スクリプトで`new`とかするとすぐGCPプロジェクトが作れる状態

## Logging
- Stackdriverにまとめて[Datadog](https://www.datadoghq.com/)を使っている。
- 全員が全部のログを見れるのは好ましくない
- サービスごとにBig Queryインスタンスが立ち上がっていて、それをStackdriverがSynkしている
- 基本的に[Datadog](https://www.datadoghq.com/)を使っている。
- Datadog + Logmaticを検証中

##  External DNS with Cloud DNS
- https://github.com/kubernetes-incubator/external-dns
- Ingressを作るとExternal DNSが自動でDNSに登録してくれる

## Heptio Ark with Cloud Storage
- https://heptio.com/products/#heptio-ark
- k8sのボリュームの中身をGCSにもろもろバックアップ

## Non GCP(GCPを使っていない部分)
- Google Container Builder使わずCircle CIでビルドしている
- もろもろに通知したり連携しようとするとCircleCIのほうがラク
- Stackdriver Monitoringは使わずDataDog。連携しやすいし
- Stackdriver Error Reportigも使わずcentory
- Gremlinを検証中。GUIでカオスな問題を発生させることができる。
  - https://www.gremlin.com/
  - [Gremlinが"Resilience as a Service"のSaaSプラットフォームをリリース](https://www.infoq.com/jp/news/2018/01/gremlin-chaos-engineering)

# GKE パネルディスカッション

覚えている内容&&印象的だったものについて

## 開発者はどこまで責任をもつんだろう？プログラム？Dockerfile?
- 会社規模感による？結構いろいろだった。

## Clusterの数どうしてる？
- こちらもいろいろだった。GoogleのBorgはワンクラスターということはそれがよいのだろうか？
- どのみちセキュリティなどでクラスター内で住み分けをするのならば、Prod/Devの住み分けもできるよね。みたいな意見も。

## データベースとかk8sにのっけている？
- メルカリは基本的にマネージドつかってる。Redisはk8sに載っているけど今日マネージドRedis出たしそっちにうつりたい。
  - [CLOUD MEMORYSTORE BETA](https://cloud.google.com/memorystore/)
- いろんな人のリポジトリ見るとk8s上にミドルウェアに載せている人多そう。
- 基本ミドルウェアは載せたくないけど、ゲームのマスターデータとかは載せちゃおうかなと思ったりはする。
- RabbitMQを動かしたいと思っている。Cloud pub/subはACQ？が短いからMQ動かしたい。

## ログとかどうしている？
- Stackdriverだけのひとたちもいる。足りない機能もあるけどGCPでまとめておきたい気持ち
- メトリクスとかまではみていないけど、CPUパフォーマンスとかくらいは見ている。
- Datadogに行く人が多い。
- Datadogに最近コンテナマップってのが出たのでそれが便利。
- クラスターが多いとDataDog便利。

## Istioをprodで動かしている？
- いなかった。。。
- カオステストはSpinnaker使ってるとカオスモンキーと連携できるし。

## セキュリティで気をつけていることは？
- クラスターレベルだとノードにログインできるのはSREだけ。
- ネームスペースごとのセキュリティをやりたい。
- kubectlたたけなくするとかは最低限。
- k8sの実運用周りのセキュリティはGKEのブログとかGKE発の情報が一番充実している。k8sブログみてもあまり実運用のセキュリティはなさそう。
- ひたすらYouTubeでGoogleの動画を見るのがいい
- PodにSSHで入られても最小限にしておくとか、GCBの脆弱性診断でコンテナーの脆弱性検知するとか。

## コンテナについて
- イメージの脆弱性もみたほうがよい。GCBの脆弱性診断はかなりよい
- コンテナーは基本軽くしたほうがよい。中身が少ない == 脆弱性も減るしデプロイ速くなる
- マルチステージビルドで軽くするのがマスト
  - https://docs.docker.com/develop/develop-images/multistage-build/

## IngressでHTTP(S) LB使ってる？
- Ingress使ってるひとがおおい。HTTPSのプロトコルの
- Ingress使うとヘルスチェックでハマる。
- デフォルトだと`/`ルートの200レスポンス。Readinessを設定しておくとそっちにGETして手探してくれる。
- あとでReadiness probe付け足してもIngressを作り直さないと反映されない…

## Cluster AutoscalerとHorizontal Pod Autoscalerとか使ってる？
- 縮退がめっちゃ遅い。。Nodeに乗ってない状態でもなくなってくれないような。
- ゲームのような一瞬でスパイクするようなサービスの場合は去年のkubeconの基調講演とか見るといいかも？YouTubeに転がっているはず。

## GKEのk8sのバージョンアップはどうしている？
- 頻繁にアップグレードして「久しぶりにアップグレードするから怖い」っていう状況を作らないのほうがよいかも。
- GKEは枯れていないのでちゃんと追従していないと振り落とされてしまう。


## CI/CDはどれ使ってる
- GCBとかJenkins。Cloud function使って通知とか出来るし（大変だけど）
- Docker enterpriseでk8s使っている人、今日はあまりいない。。。
- Spinnakerのエラーは辛い。halコマンドで入れていればエラーログが多少見えるけど、Javaのエラーが読める人がいないと辛い。


## AppEngineのようなバージョンURLの仕組み。どう用意している？
- Istioでやる予定。
- Istioで出来るならIstioの事例待つ…deeeetさんはよ…
- ゲームだとクライアントアプリ2バージョン対応しないといけないので、クライアントアプリ側でバージョンURLとか分けたいかも。

## Jenkins Xってどうなの？
- 人柱もとむ。

## 木曜深夜2時にやっているKubernetes Community Meeting参加している人？
- https://kubernetes.io/community/
- よくわからん。謎だ。

## GKE以外のマネージドk8sサービスを追いかけている人？
- AWS Ffagateが実現するとすごいけど、どうなるのかなーー。

## Node Poolどうしている？
- バージョンとかでノードプール分けるならばブルーグリーンすればよいのでは
- ハードが限定されるとかマシンレベルで事情があるときにノードプールで分けるとか。


# 感想
Kubernetesの関連・周辺エコシステムや運用についての知見が盛りだくさんだった。  
SpinnakerやIstioは名前を聴くけどまだあまり運用実績は日本になさそう？  
k8sだけではなくて、大規模システムのマイクロサービス化について非常に参考になった。  
個人で遊んでいるとRBACとかただ難しいだけなんだけど、ベンダーサービス(IAM etc)に依存せずにk8s上だけでセキュリティ構築するのには重要になっていくのかな？



# 関連
- [GCPUG Tokyo Container Builder Day February 2018 #gcpug 参加メモ](/2018/02/05/gcpug-container-builder-day/)
- [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)


