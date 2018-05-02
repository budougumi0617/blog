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
今回は有志による一人10分のCI/CDツールの紹介。自分は発表者としても参加させていただいた。

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
  - ※ @cyberblack28さんはRancherのほうからいらっしゃった方


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


# 会場アンケート
会場のアンケートによると普通のJenkinsを使っているところが多いようだった。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">cndjp第5回で実施した、使っているCI/CDツール アンケートの集計結果です。ご査収ください。<a href="https://twitter.com/hashtag/cndjp5?src=hash&amp;ref_src=twsrc%5Etfw">#cndjp5</a> <a href="https://t.co/xOTZx5Patd">pic.twitter.com/xOTZx5Patd</a></p>&mdash; Hayakawa Hiroshi (@hhiroshell) <a href="https://twitter.com/hhiroshell/status/990967880220000256?ref_src=twsrc%5Etfw">2018年4月30日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Jenkinsも良いとは思うんだけど、(別のCIサービスを使っても結局"XXXXおじさん"に頼ることになるので、)Jenkinsおじさん問題より、オンプレCI環境のリソースの限界を感じる。  
リース切れ待ちの放置PCとか大量にスレーブにしたりしてリソース確保しているのだろうか？（実体験）  
TravisCIが少なかったのが意外だった。


# 感想
一日で9種類のCIサービスの概要を聞けるお得な勉強会だった。自分でちゃんとYAMLを書いて設定したことがあるのはCircleCIくらいだったので非常に勉強になった。  
とくにJenkins XやMicroclimateなどの最新情報も聞けたので（自分は除いて）登壇者のみなさんの情報力に感謝するしかない。
YAMLで設定を書くのはもはやデファクトのようだけど、記載内容も共通化されていくと引越しが簡単で嬉しい。  
TeamCityやWindows系のCI(AppVeyor, VSTS(TFS))はなかった。あまりニーズがないのか、やはりWindowsでビルドするような案件はオンプレでCIしたいのか？  
自分はVSTS, AppVeyor、TravisCI、CircleCIなどをよく使ったけど、いま個人で新しいリポジトリ作るときは基本CircleCIな気がする。  
「デプロイ後ロールバックして前回デプロイに戻せるのか？」とか全部のツールに共通の質問とかしてみればよかった。  


# 関連
- [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)
- [Cloud Native Developers JP 第1回勉強会 参加メモ #cndjp1](/2017/11/23/cndjp1/)
- [[k8s]Cloud Native Developers JP 第2回勉強会 参加メモ #cndjp2](/2017/12/18/kubernetes-in-production-cndjp2/)
- [[k8s]Cloud Native Developers JP 第3回勉強会 #cndjp3 参加メモ](/2018/02/03/kubernetes-with-availability-cndjp3/)
- [Kubernetes Network Deep Dive!（Istioもあるよ） cndjp第四回参加メモ #cndjp4](/2018/04/01/kubernetes-network-deep-dive-cndjp4/)

# （余談）登壇に対するKPI
今回、初めて社外発表した。「発表時間10分」というのは初めて社外発表する身としては非常にありがたい機会だった。  
Spinnakerは前から調べようと思っていたが、使ったことがなかったので今回の勉強会をデッドライン駆動の条件として動かすところまで出来た。  
使い込んでいたわけではないので、薄い内容になってしまったが、「動いているところを見たことある人」は0人だったので、それなりに情報提供は出来たかなと思う。

## 良かったこと
- 発表時間を守ることができた
  - 正直一度も練習出来ないままぶっつけ本番だったので納まってよかった。
- 一度ブログ記事にアウトプットした内容など、資料に落としやすかった
  - 出し惜しみせずにブログにしていたので資料の構成作りなどラクにできた
- 主催者のhhiroshellさんが大きな時計を設置してくださっていたので、時間確認しながら進めることができた
- 事前入りして接続チェックすることができた
  - 最初に用意されていたアダプタでは映らなかったのでちゃんと確認しておいてよかった。純正最強説。
- 動くところを見せることができた
  - デモ環境を用意するして大まかな機能の紹介をすることが出来た


## だめだったところ
- 一連の流れをデモすることは出来なかった
  - コンテナビルドから始めたのでトリガー→承認までの流れをデモできなかった
  - マニュアル開始で良かった
- そもそも一度も発表練習しないで本番に臨んでしまった
  - 言い訳をすると当日にデモ環境壊してしまって、練習するつもりだった時間にk8sクラスタ5回くらい作りなおしていた。。。
- もうちょっとデモの時間多くしてもよかったかもしれない
  - 調べれば出てくる文字情報より動くもの見せる時間多くしたほうがよかったかもしれない

## 次回に向けて
- 資料作成やデモ環境の作成はある程度諦めてしっかり発表練習の時間を作る
- 発表前だからといって出し惜しみせずに調べたことは一度まとめてブログ化しておく
- 資料作成はある程度は時間ベースで区切る
  - キリがなくなるのでほどほどにする

発表してみて、いろいろな方に声かけていただけたし、最後の会場アンケートで「Spinnkaer使ってみたいと思った」人が数名いらっしゃったので非常に良い経験になった。  
5月も発表機会があるので引き続き登壇してみようと思う。  
聞いてくださった皆様ありがとうございました。
