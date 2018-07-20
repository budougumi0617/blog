+++
title = "LINE Developer Meetup in Tokyo #40 -Kubernetes-参加メモ #LINE_DM #k8s"
date = 2018-07-20T13:32:00+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["report", "kubernetes"]
tags = ["spinnaker", "line", "k8s","rancher"]
+++




LINE Developer Meetup in Tokyo #40 -Kubernetes-に参加してきたのでメモ。  
オンプレでのk8sの運用や、Rancher、SpinnakerやDruckerといったツールについて学ぶことができた。

<!--more-->

- https://rancher.com/
- https://www.spinnaker.io/
- https://github.com/drucker

|||
|---|---|
|URL|https://line.connpass.com/event/92049/|
|会場|LINE株式会社(新宿ミライナタワー)|
|日時|2018/07/18(水) 19:00 〜 20:50|
|ハッシュタグ|[#LINE_DM](https://twitter.com/hashtag/LINE_DM)|

# 所感
クラウドベンダー上ではなくオンプレでKubernetesを運用する事例が多かった。

- 自分はPaaSを使った運用しか知らないので、オンプレ運用はシステム的にも、運用体系の構築的にも大変そうだと印象を受けた
  - 主要なマネージドクラウドに対応しているOSSは多いが、自分たちのオンプレ環境でそのようなOSSを使いたかったら自分たちでプラグインとか作るのだろうか？k8s対応がされているOSSはk8sのAPI経由で利用できるのかな？
- Rancher2.0のアーキテクチャについて聞くことができたので、聞いた内容を参考に自分も少しコード読んでみようと思う
- Spinnkaerは個人的に少し触ったくらいの知識しかなかったので、本番運用での懸念点などを聞くことができて良かった
- 結構どこもk8sのカスタムリソース（カスタムコントローラ）を作っているようなので、自分でも簡単な定義を作ってみようと思った
  - そもそもまずk8sのAPIを一通り叩くところからなのだが。。。
- 勉強会で生ビール出るLINEさんの気合い(Meetupだから？)


以下、当日のメモ。


# Rancher 2.0 Technical Deep Dive
西脇 雄基（LINE)

- Rancher 2.0 Technical Deep Dive
  - https://www.slideshare.net/linecorp/rancher-20-technical-deep-dive/

## Verda is Kubernetes as a Service
- LINE内でk8s as a ServiceのVerdaというPrivate Cloudを構築している
  - 利用状況は600ノード以上
  - 台湾、韓国、ベトナムなどからも利用している
  - 18年5月から垂直立ち上げを行なう必要があった
  - プロダクションレディではない。開発環境で評価中
- Verda利用者はVerdaの独自API定義でk8sクラスターを操作するが、垂直立ち上げしないといけないのでLancher2.0で内部処理を行なっている
  - ユーザーにはVerdaのAPIしか見せていない。いずれRancherは置き換えるかもしれないから。

## Lancher2.0
- https://rancher.com/
- k8sのオペレーションがわからなくてもRancherのGUIからいろいろできる。
- 2.0になるにあたり、Goでフルスクラッチされたのでコード量が少なくなって読みやすくなった
- 主に３つの機能で構成されている
  - Rancher Server
      - k8s上で動く必要がある
  - Rancer Cluster Agent
      - クラスターにひとつ
  - Rancher Node Agent
      - ノードごとに一つ動いている
- Rancherで設定したデータは全てk8s上に保存される
- デプロイはRancherのカスタムコントローラが行なっている
- Websocket経由でエージェントを操作する
- エージェントはTCPプロキシサーバのように動いている
- 殆どの操作はコントローラでRancher Server駆動でで動く

## Rancher2.0 API
- おおまかに5つのグループに分けることができる
  - Management API 
      - ユーザー操作、GUIで使う。
      - k8sのリソースに変換して操作が行われる
  - Auth API
      - 一般的な認可認証
  - k8s Proxy API
      - デプロイとかにつかう
      - Rancher管理のk8sは原則このAPI経由で操作される
      - Rancherユーザとして認証が行われると、Agentのサービスアカウントを使って操作が実行される
  - Dialer API
      - k8s上のAgentがwebsocketセッションを作る
      - エージェント用のAPIはユーザー用のAPIで別のトークンが使われる
  - RKE Node Config API
      - Rancher Kubernetes Engine
          - https://github.com/rancher/rke
      - ノートの設定に応じてコンテナの起動などを行なう

## Rancher2.0のコントローラ
- だいたい4種類のコントローラがある
  - API Controllers
      - API定義が叩くコントローラ。リソース宣言の変更
  - Management Controllers
      - クラスターの起動や後述の2種類の低レイヤーコントローラの起動
  - Cluster(User) Conrtrollers
      - 実際にリソースの監視や生成を行なうコントローラ
  - Workload Controllers
      - k8sにないRancherが必要とする機能のk8s拡張コントローラ

## Rancher2.0の良かった点
- 動作環境に対する制約が少ない
- k8sを管理する上で興味深いカスタムコントローラがある
- Norman frameworkというAPI frameworkが良い感じ
  - https://github.com/rancher/norman

## Rancher2.0の難しい点
- ドキュメントが少ない。RKEもNorman framworkも少ない
- Rancher Server(Rancher本体)は複数デプロイできなそう。
- Rancherは1バイナリなので、もし一部機能でも負荷があがるとヤバそう。
- モニタリング機能がまだ不十分に感じたので、そこは別途・あるいは自前で用意する必要がある

## Verdaの今後
  - 当面はRancher2.0で9月のリリースする予定
  - スケーラビリティのための拡張やモニタリングの強化を検討中

# Kubernetes-as-a-Service in Yahoo! JAPAN Deep Dive
村田 俊哉（ゼットラボ)

- Kubernetes as a Service in Yahoo! JAPAN Deep Dive 
  - https://speakerdeck.com/shmurata/180715-line-meetup

## Yahoo!のKubernetes as a Service
- Yahoo!向けのk8s as serviceの開発をしている
- 2016/07から開発を始め、2017年10月からYahooの一部のサービスでプロダクションで利用している。
- セルフサービス
  - 開発者が自由にクラスタを作成・削除できる
- マネージド
  - ゼロダウンタイムでアップグレード
  - 社内モニタリングサービスと連携
- スケーラブル
- k8sのオペレーションから運用者を開放する
- 類雑なk8sクラスタオペレーションを機械化する

## KaaSの要件
- スケーラブル
- 非同期モデル
- 堅牢性
- これらの要件を満たす複雑な分散システムの基盤を構築する必要がある

## Kubernets as a Service on Kubernetes
- 分散システム基盤としてのk8s自体を利用する
- k8sはAPIを拡張したりすることもできる。
- Custom Resource Definitions(CRD)
  - https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/
- k8s Operatorを内製
  - カスタムリソース・カスタムコントローラで運用ノウハウを実装する
  - CRDは宣言的に書けるので命令的な実装より堅牢にできる
  - Reconciliation Loop
    - 現在の状態(status)を望ましい状態(spec)に近づける。
    - specとstatusを比較してその差分をなくすようにコントローラが動く

## Kubernetes as a Serviceのメリット
- k8sのクラスターをゼロダウンタイムでアップグレードするのは難しい
  - ワーカーノードをどう更新する？
- クラスターのアップグレードを実現する上で注意したこと
  - ローリングアップグレード
  - アップグレードが失敗するようなら途中で停止するようにする
  - 途中で停止した場合はロールバックまたは再開を容易に行われるようにする
- なぜ頻繁に更新が行われる状態が嬉しいのか
  - アップグレードのコストが少ないのは重要
    - 変更しやすいしセキュリティやパフォーマンスの改善が見込める
  - 月に一回程度ローリングアップグレードができてる

## まとめ
- 運用ノウハウをソフトウェアとして実装し、ソフトウェアに仕事をさせる
- k8sをフレームワークのように利用することで複雑な分散システムを簡単に実装

# SpinnakerによるContinuous Delivery
中島 大一 [@deeeet](https://twitter.com/deeeet)（メルカリ)

- Continuous Delivery for Microservices with Spinnaker at Mercari
  - https://speakerdeck.com/tcnksm/continuous-delivery-for-microservices-with-spinnaker-at-mercari

## Why microserveies?
- 大規模開発でモノリスは大変

## Continuous Delivery(CD)
- すばやく価値提供することですばやくフィードバックを得る
- 小さくリリースし続ければもしものときも対応がたやすい
- リリースをストレスなくできれば開発に集中できる

## CDの課題
- どうやってCDをマイクロサービスでやるか
- どうk8sにデプロイするか
- モノリスのときは(数も少ないし)SREだけでもできた

## MicroserviceのContinuous Delivery
- 開発チームが自分たちでリリースするのが当たり前
- `kubectl apply`の機能だけでは不十分
  - applyしたらそのまま。デプロイ後のチェックもしてくれないしCIからロールバックもできない。
  - カナリーリリースも難しそう
- 開発者がデプロイパイプラインをカスタマイズできて各Microserveiceで最適化されたk8s上へのデプロイができること

## Spinnaker
- https://www.spinnaker.io/
- マルチクラウドでk8sにデプロイできる
- 「マイクロサービスアーキテクチャ」でも言及されるNetflixのノウハウが詰まっている
- 最近ではSpotifyも使いはじめた
  - https://www.ciodive.com/news/how-spotify-is-migrating-from-an-in-house-docker-orchestration-platform-to/525465/

## How to use
- 基本的には3種類の機能を組み合わせてパイプラインを作る
  - Stage
      - デプロイ・ロールバック処理などの基本的な処理
  - Trigger
      - いつパイプラインを実行するのか
      - Gitやコンテナがプッシュのタイミングで実行
  - Notification
      - Slackやメールに通知を飛ばす
      - 自動で全て行なうからこそ通知で状況を確認できるようにする

## Continuous Deliveryを成功させるためには
- 恐れずデプロイをできること
- safeguardsがちゃんとあること
    - たとえばデプロイ禁止時間の設定とか
    - 簡単にロールバックできる etc
- Spinnakerはk8sのオペレーションが完了したかを確認してくれる
    - (コンテナが起動した、ではなく)Podがちゃんとレディになったとか

## Spinnaker at Mercari
- 2017年から使っている
- k8s V1 Providerでデプロイしている
    - https://www.spinnaker.io/setup/install/providers/kubernetes/
- 30アプリがデプロイしている
- Spinnaker自体をSpinnaker専用のGKE Clusterに展開している
- 1つのSpinnakerで世界各極のサービスをデプロイしてる

## Spinnakerのよいところ
- 管理者として
  - 簡単に開発者がデプロイすることができる
  - いろいろ抽象化ができる
  - いずれはメルカリ専用のステージを提供できたらなーということをしている
- 開発者として
  - 簡単にカスタマイズパイプラインをつくれる
  - 他のチームのパイプラインの参照もできる
  - GUIでできる

## Spinnakerの悪いところ
- 管理者として
  - 不安定すぎる
  - (Spinnaker自体を管理する)Halyardがちょっと大変
      - コマンドベースなので設定変更時レビューも大変
  - デプロイ先のクラスターがひとつ落ちたとき他のクラスターにも影響が受けやすい
  - 何か起きたときはSlackやGitHub Issueでいろいろ聞かないといけない
- 開発者として
  - GUI難しい
  - 変更するとき怖い
  - Netflixは9,000パイプラインを全部GUIで編集していたらしいけどどうやってるんだろう

## Spinnakerの今後どう使っていくか
- 今後はKubernetes Provider V2でデプロイしていく
  - https://www.spinnaker.io/setup/install/providers/kubernetes-v2/
- DCD(Declarative Continuous Delivery )を使ってGUIはRead Onlyにしたい
  - Codifying your Spinnaker Pipelines
  - https://blog.spinnaker.io/codifying-your-spinnaker-pipelines-ea8e9164998f
- ACA(Automated Canary Analysis)使いたい
  - Automated Canary Analysis at Netflix with Kayenta
  - https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69


# Clovaにおける機械学習モジュールの配信＆運用基盤の紹介
服部 圭悟 [@keigohtr](https://twitter.com/keigohtr)（LINE)	

- Clovaにおける機械学習モジュールの配信＆運用基盤の紹介
  - https://www.slideshare.net/linecorp/clova-106423895

## Clovaの紹介
- LINEのAIスピーカー
  - https://clova.line.me/
- LINEが使える
- 音質が良い
- バッテリー搭載
- 赤外線標準装備
- 最近スキル開発も開始

## Clovaにおけるk8s活用事例
- 機械学習が流行っている
- 環境も整ってきた
- ただ、今まで注目されていたのは学習環境のツール
- 機械学習の運用フェーズを担うOSSはそんなにまだない
- OSSとしてDruckerを作成した(クローズドベータ中)
  - https://github.com/drucker

## Drucker
- https://github.com/drucker
- 機械学習の配信フレームワーク
  - 学習モデルの配信を簡単に
  - 学習モデルの管理と運用を簡単に
  - 既存のシステムへの統合を簡単に

## Druckerの特徴
- High Avalialbility
  - gRPC x Microservice
  - テンプレートに沿ってアルゴリズムを配信できる
  - proto定義などはだいたい定義済みのprotoを使えばOK
- Management
  - ダッシュボード(WebGUI)から学習モデルをアップグレードできる
  - 学習モデルのバージョニング
  - モデルの性能を測定・可視化など
- Integration
  - Drucker Clientで任意の言語で接続可能

## Kubernetes via Rancher
- Rancher1.6ベースでk8s上に展開している
  - Auto Healing
  - Auto Acaling
  - Rolling update

## その他の運用
- Serverのsetup
  - staging/productionなどはk8sのnamespaseなどで分けている
- ロードバランシング
  - Load balancingはgRPCが使えnghttpx ingress controller
      - https://github.com/zlabjp/nghttpx-ingress-lb
- AB test
  - Druckerのダッシュボードからk8sにMLサービスを任意の設定を起動できる
  - Drucker自体に評価機能はないので、fluentd-k8s, kibana + ElasticSearchを使ってログ分析をしている
      - https://github.com/fluent/fluentd-kubernetes-daemonset

## まとめ
- Drucker + k8sでMLサービスの管理を簡単にできた
- Druckerを使ってWebGUIから機械学習モデルを簡単に配信・更新
- Kubernetesでサーバ運用を自動化
- ABテストなどもラクにできるようにできた


# 関連
- [Tag: Spinnaker](/categories/spinnaker/)
- [Spinnakerを使ったカナリーリリースの自動評価 #spinnaker #kayenta](/2018/06/29/get-started-auto-canary-analysis-by-spinnaker-with-kayenta/)

