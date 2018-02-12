+++
title= "[Go]Google Kubernetes Engineのチュートリアルをやるときに注意すること"
date= 2018-02-12T08:04:35+09:00
draft = false
slug = ""
categories = ["gcp"]
tags = ["gke","k8s","gcp"]
author = "budougumi0617"
+++

Google Cloud Platform(GCP)のKubernetes Engine(GKE)でKubernetes(k8s)を動かしたいと以下のチュートリアルを始めた。

**Container Engine での Go Bookshelf の実行**  
https://cloud.google.com/go/tutorials/bookshelf-on-container-engine?hl=ja

**GoogleCloudPlatform/golang-samples**  
https://github.com/GoogleCloudPlatform/golang-samples

2018/02/12時点で以下の点に注意することがあったので、メモしておく。

# TL;DR
- 日本語版の`gcloud`のインストールページは最新ではない
- `dep`コマンドなどは使わず、`go get -u ./...`で依存パッケージを取得する
- クラスターを作成するときは`--cluster-version`でk8sのバージョンを最新にしておく


# 日本語版の`gcloud`のインストールページは最新ではない
チュートリアルの「始める前に」の途中にリンクしてあるgcloudのインストールページは最新版の`gcloud`へのリンクではない。  
（`180.0.0`のtarが書いてあるが、2018/02/12時点で最新は`188.0.0`）  
以下のコマンドで最新版の`gcloud`コマンドが取得できる。

```bash
$ curl https://sdk.cloud.google.com | bash
```

# `dep`コマンドなどは使わず、`go get -u ./...`で依存パッケージを取得する
チュートリアル用の`golang-samples`リポジトリは`glide`や`dep`のような依存管理ツールの設定がされていない。  
逆にいうと、常に最新の依存ライブラリに依存している。  
2018/02/12現在`dep`コマンドで依存関係を解決すると、`satori/go.uuid`リポジトリなどの破壊的変更が取り込めないので、  
ビルドエラーになってしまう。(Dockerでビルドしてもvendorディレクトリが取り込まれてしまうためエラーになる）  
ローカルで依存関係を解決したいときは`go get`コマンドで行うこと。

```bash
$ cd $GOPATH/src/github.com/GoogleCloudPlatform/golang-samples/getting-started/bookshelf
$ go get -u ./...
```

# クラスターを作成するときは`--cluster-version`でk8sのバージョンを最新にしておく

以下のコマンドで利用できるk8sのバージョンとデフォルトが確認できる。

```bash
$ gcloud container get-server-config
```

もしここで`defaultClusterVersion: 1.7.XX-gke.1`と表示されたなら、変更するほうがよい。  
2018/02/12現在以下の問題があり、1.7.X系はStackDriver周りでエラーが出るので変更しておく必要がある。

**Error while sending request to Stackdriver ... unsupported protocol scheme**  
https://github.com/GoogleCloudPlatform/k8s-stackdriver/issues/92

チュートリアルの「Container Engine クラスタの作成」に書いてある`gcloud container clusters create`コマンドに  
クラスターバージョンを指定するオプションをつけてクラスターを作成する。

```bash
$ gcloud container clusters create bookshelf \
  --scopes "cloud-platform" \
  --num-nodes 2 \
  --cluster-version 1.9.2-gke.1
```

指定できるバージョンは`gcloud container get-server-config`コマンドでも確認できたが、ドキュメント的には以下に記載してある。

**Versioning and Upgrades**  
https://cloud.google.com/kubernetes-engine/versioning-and-upgrades


# 終わりに
サンプルリポジトリである[GoogleCloudPlatform/golang-samples](https://github.com/GoogleCloudPlatform/golang-samples)自体も
定期的に更新されているので、何か問題があったら一度pullするだけで解決するかもしれない。  
実際最初PubSub周りの疎通が出来ていなかったのだが、サンプルリポジトリをpullして`go get -u ./...`することで解決した。

最初、`dep`コマンドで依存関係を解決したため、ビルドエラーを出していた。  
PRチャンスかと思ってPRを出し、CIの結果を見て自分が間違えていることに気づいた。  
自分が間違えていたのですぐPRは閉じたのだが、メンテナーの方が優しくコメントしてくださってすごいありがたい気持ち。

https://github.com/GoogleCloudPlatform/golang-samples/pull/390#issuecomment-364297421


GKE自体は動かせたので、次はgRPCをGKEで動かしてみる。


# 参考
**Container Engine での Go Bookshelf の実行**  
https://cloud.google.com/go/tutorials/bookshelf-on-container-engine?hl=ja

**GoogleCloudPlatform/golang-samples**  
https://github.com/GoogleCloudPlatform/golang-samples

**GCPのgcloudコマンドをインストールする**  
https://qiita.com/kentarosasaki/items/2232113b44b016a56adc

**Error while sending request to Stackdriver ... unsupported protocol scheme**  
https://github.com/GoogleCloudPlatform/k8s-stackdriver/issues/92

**Versioning and Upgrades**  
https://cloud.google.com/kubernetes-engine/versioning-and-upgrades
