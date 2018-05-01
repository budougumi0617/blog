+++
title= "[k8s]あつまれ！ CI/CDツール大集合！ - cndjp第5回 参加メモ #cndjp5"
date= 2018-05-02T09:00:41+09:00
draft = false
slug = ""
categories = ["report","kubernetes","spinnaker"]
tags = ["k8s","cndjp", "spinnaker", "docker"]
author = "budougumi0617"
+++

Cloud Native Developers JP(cndjp) 第5回勉強会に参加してきたので、自分メモ。  
今回は有志によるCI/CDツールの紹介。自分は発表者としても参加させていただいた。

|||
|---|---|
|URL|https://cnd.connpass.com/event/84310/|
|会場|オラクル青山センター 13F セミナールーム|
|日時|2018/04/27(金) 18:30 〜 21:30|
|資料|https://cnd.connpass.com/event/84310/presentation/|
|ハッシュタグ|[#cndjp5](https://twitter.com/hashtag/cndjp5)|


# Spinnaker入門
[@buougumi0617](https://twitter.com/budougumi0617)  
https://speakerdeck.com/budougumi0617/introduction-spinnaker

- https://www.spinnaker.io
- SpinnkaerはNetflixやgoogleが開発している継続的デリバリーツール
- k8sクラスターに対してカナリーリリース、ロールバックなどが簡単に行える
- カナリーリリースのスコアを可視化してしデプロイの自動化をする自動カナリー分析もできる
- なおスライド中の参考リンクは以下にまとめてある
  - [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)

# CircleCI first-step
[@hisa9chi](https://twitter.com/hisa9chi)  
https://speakerdeck.com/hisa9chi/circleci-first-step

- https://circleci.com/
- クラウド上で提供されるCIサービス
- オンプレで構築できるエンタープライズ版もある
- Linux、McOS上でのビルド(+オンプレ)をサポート
- 最近CircleCI 2.0となりコンテナ上で実行されるようになった
- 単一ジョブとワークフローを定義可能
- クラウド上で実行したあとSSHログインしてビルドエラーの調査などもできる
- 柔軟な通知設定や、並列実行、ビルド情報のcacheを使うこともできる
- (制限もあるが、)Local CLIを使えばクラウドで実行する前にローカルで動作確認も可能


# AWS CodeBuild Overview
[@sakajunquality](https://twitter.com/sakajunquality)  
https://speakerdeck.com/sakajunquality/aws-codebuild-overview

- https://aws.amazon.com/jp/codebuild/
- AWSのフルマネージドなCIサービス
- AWS Code Pipelineのビルド作業を担っている
- AWS上の認証情報やIAM権限を使える
- ビルドイベントの通知はAWS SNS、Lambdaを経由して受け取れる
- ビルドログはCodebuild ConsoleやCloud Watchから

# Concourse入門
[@watawuwu](https://twitter.com/watawuwu)  
https://speakerdeck.com/watawuwu/concourse-getting-started

- https://concourse-ci.org/
- Pivotal中心にGoで開発されているCI/CDのOSSツール
- パイプライン中の各ジョブをYAMLで定義する
- パイプラインの進捗表現などビジュアル面がよい
- 基本的にビルドはコンテナ上で実行される
- Web UIベースで操作をするが、CLIからも同等の実行できる


# Google Container Builder Overview
[@sakajunquality](https://twitter.com/sakajunquality)  
https://speakerdeck.com/sakajunquality/googlecontainerbuilderoverview

- https://cloud.google.com/container-builder
- GCP上でコンテナーをビルド出来る。
- ビルドはYAMLで定義する
- ビルドステップは毎回別々のコンテナ上で行われる
- GCP上の認証情報やIAM権限を使える
- googleがビルドに使うコマンドを含んだコンテナを公開している
  - https://github.com/GoogleCloudPlatform/cloud-builders
- ビルドイベントの通知はGoogle Cloud Pub/Sub経由で
- ビルドログはStackdriver経由で


# Werckerの紹介
kosuke-

- http://www.wercker.com/
- Travis CIやCircleCIのようなCI Service
- パイプラインの概念があり各アクション（ステップ）をYAMLで定義する
- Oracle傘下なのでOrace Cloud上のk8sに簡単にデプロイが可能
  - https://docs.oracle.com/cd/E83857_01/iaas/wercker-cloud/wercm/kubernetes.html
- Wercker CLIを使うとローカルでビルドパイプラインの検証ができる

# Jenkins X on GKE & Rancher2.0 on ORACLE Cloud
[@cyberblack28](https://twitter.com/cyberblack28)  
https://www.slideshare.net/cyberblackvoom/jenkins-x-on-gke-rancher20-on-oracle-cloud-95230016

- http://jenkins-x.io/
- k8s上で構築するJenkins
- `jx`コマンドを起点としてJenkins環境をk8s上に構築できる
  - http://jenkins-x.io/getting-started/install/
- `jx create cluster gke`というようにコマンドを実行すると、あとは選択肢を選んでいくだけでJenkinsが構築できる
- Rancher2.0 betaを使ってOracle Cloud上に構築したRancherからGKE + Jenkins Xクラスターへの接続も出来た
  - ※ @cyberblack28-sanはRancherのほうからいらっしゃった方


# GitLabによるComplete DevOpsの実現
[shkitayama(@spchildren)](https://twitter.com/spchildren)  
https://speakerdeck.com/shkitayama/gitlabniyorucomplete-devops

- https://about.gitlab.com/
- DevOpsに必要なサービスを全て統合させた総合サービス
- サービス開発の計画フェーズのプランニングからissueトラッカー、CI/CD、運用フェーズのモニタリングまでトータルでフォローする
- GKEも統合していてk8sを使ったデプロイも簡単にできる。
  - [GitLabがGoogleのKubernetes Engineを統合、コンテナアプリケーションのデプロイが超簡単に](https://jp.techcrunch.com/2018/04/06/2018-04-05-gitlab-launches-a-native-integration-with-googles-kubernetes-engine/)
- あえてネガティブなことを言うとGUIより機能重視の面はある
- GKEとのインテグレーションのプロモーションでGCPが$500分使えたりもする(通常のGCPお試し$300 + GitLabボーナス$200)
  - [GitLab + Google Cloud Platform = simplified, scalable deployment](https://about.gitlab.com/2018/04/05/gke-gitlab-integration/)
- コミュニティも活発
  - https://gitlab-jp.connpass.com/


# 「Microclimate」
[@capsmalt](https://twitter.com/capsmalt)  
https://www.slideshare.net/capsmalt/cndjp-microclimateby-capsmalt

- https://microclimate-dev2ops.github.io/
- IBM製のアプリケーション開発環境
- オンプレ環境にインストール or k8s上にHelmで展開
- テンプレからk8s上でのJava, Node, Swift製のマイクロサービスの開発を開始可能
- Web IDE搭載で動的ビルド・テストが可能
- 無償お試し版を任意の環境にインストールしてトライアルすることも可能
  - [IBM Cloud Private: Kubernetesをオンプレミス(IaaS)に導入してみる](https://qiita.com/capsmalt/items/d15055ab3cb423d2d7ae)
- オンブラウザで試せる環境もある
  - https://ibm-dte.mybluemix.net/ibm-cloud-private

# (番外)Skaffoldを使ってみた
[@hhiroshell](https://twitter.com/hhiroshell)  
https://speakerdeck.com/hhiroshell/skaffoldwoshi-tutemita

- https://github.com/GoogleContainerTools/skaffold
- ローカルのファイル変更をトリガーにコンテナのビルドからk8sへのデプロイを自動的に行ってくれる。
- build/push/deployの各フェーズをプラガブルに変更可能
  - pushはGCR、deployはhelm、とか
- `skafolld dev`をローカルで起動しておけばコンテナの依存ファイルの変更を検知してホットリロードされる

# 感想

# 関連
- [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)
- [Cloud Native Developers JP 第1回勉強会 参加メモ #cndjp1](/2017/11/23/cndjp1/)
- [[k8s]Cloud Native Developers JP 第2回勉強会 参加メモ #cndjp2](/2017/12/18/kubernetes-in-production-cndjp2/)
- [[k8s]Cloud Native Developers JP 第3回勉強会 #cndjp3 参加メモ](/2018/02/03/kubernetes-with-availability-cndjp3/)
- [Kubernetes Network Deep Dive!（Istioもあるよ） cndjp第四回参加メモ #cndjp4](/2018/04/01/kubernetes-network-deep-dive-cndjp4/)



