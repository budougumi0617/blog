+++
title= "GKEでSpinnakerを使ったKubernetesの継続的デリバリーを試してみた"
date= 2018-04-24T21:01:54+09:00
draft = false
slug = ""
categories = ["Spinnaker", "kubernetes"]
tags = ["Spinnaker", "CD","k8s", "gke"]
author = "budougumi0617"
twitterImage = "logos/spinnaker.png"
keywords = ["Spinnaker", "Continuous Delivery", "Kubernetes", "k8s", "GKE"]
+++

Google Kubernetes EngineとSpinakerを使ったkubernetesの継続的デリバリーを試してみた。

Spinnakerを始めるにあたってまずGCPでサンプルの継続的デリバリーを試してみた。  
一通りのコピペでGoogle Kubernetes Engine(GKE)に対するSpinnakerを使った継続的デリバリーを試せる。

**Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine**  
https://cloud.google.com/solutions/continuous-delivery-spinnaker-kubernetes-engine


![Spinnakerのパイプライン](/2018/04/spinnaker-pipeline.png)

なお、日本語のページの手順と英語のページで手順に差分があるので、基本英語のページで見たほうが良さそう。

**Spinnaker と Container Engine を使用した継続的デリバリー パイプライン**  
https://cloud.google.com/solutions/continuous-delivery-spinnaker-container-engine?hl=ja

日本語版の手順は`clusterrolebinding`などの設定が抜けているので、
GKEに1.8系のk8sクラスターを立てているとRBACで疎通できない気がする。

# TL;DR

- GCPのハンズオンを使ってSpinnaker + GKEを試した
  - https://cloud.google.com/solutions/continuous-delivery-spinnaker-container-engine
  - Spinnakerによる継続的デリバリー環境の構築作業
  - GKE上にk8sクラスターを作成
  - Helmを使ってk8sクラスタ上にSpinnakerをインストール
  - JSONベースでパイプラインを設定する
  - Google Container Builder(GCB)でビルドトリガーを設定しておく
- 構築した継続的デリバリー(Pipeline)の内容
  - コンテナがGCBでビルドされるたびにSpinnakerのDeploy pipelineが実行される
  - カナリーリリースが行われ、パイプラインが手動承認のステージで停止する
  - カナリーリリースされたコンテナにアクセスして動作確認することが出来る
  - 動作確認後、手動承認を行うとパイプライン処理が再開し、プロダクションにデプロイされる


# ハンズオンで構築する環境について
ハンズオンは簡単に言うと以下の構築を行う

- GKE上にk8sクラスターを作成
- Helmを使ってk8sクラスタ上にSpinnakerをインストール
- JSONベースでパイプラインを設定する
- Google Source Resistory(GSR)にデプロイ対象のリポジトリを用意する
- Google Container Builder(GCB)でビルドトリガーを設定しておく

Spinnakerのインストール方法はいくつか方法があるが、このハンズオンではHelmによるインストールを行っている。
また、パイプラインの定義自体はJSONで定義した既定の設定を用いる。


# ハンズオンで試せる内容
ハンズオンで構築したパイプラインを使うと、
Google Source Repositoryで`git tag`したコードがGoogle Container Builderのトリガーで自動的にビルドされる。
その後、Spinnakerによって自動カナリーリリースが始まる。

![Spinnkaerによるデプロイ](/2018/04/spinnaker-deploy.png)

カナリリリースが終わると、デプロイパイプラインは「承認確認フェーズ」で一度停止する。

![Spinnkaerによってデプロイされたカナリーコンテナ](/2018/04/spinnaker-canary.png)

カナリーデプロイされたコンテナにアクセスして正しく動いているか確認できる。

![Spinnakerのパイプライン](/2018/04/spinnaker-pipeline.png)

動作確認後、パイプライン上から承認をすると、全てのプロダクションコンテナへ更新が反映される。


# 終わりに
ざっくりだがGKE上のSpinnakerのハンズオンを試してみた。
継続的デリバリーツールについては実際になにか運用してみないとメリットがわかりにくいので、
一連の継続的デリバリーまで体験できるのは非常によかった。
ただ、この構成だとどうやって`Kayenta`を有効にするのかわからないので、できたら`hal`コマンドで構築してみたい。


なお、ご自分のGCPアカウントで行った場合はもろもろをクリーンアップしないと課金がされ続けるので注意する。

https://cloud.google.com/solutions/continuous-delivery-spinnaker-container-engine#clean-up


# 関連
[Spinnakerの始め方いろいろ](/2018/04/22/how-to-start-spinnaker/)


