+++
title = "SpinnakerでGKEの自動カナリー分析をするときはStackdriver k8s Monitoring V2 BETAを使う #spinnaker #kayenta"
date = 2018-06-27T07:48:57+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["spinnaker"]
tags = ["spinnaker", "kayenta"]
+++


[Spinnaker](https://www.spinnaker.io/)とKayentaを使えばカナリーリリースの結果を自動評価することができる。

**Automated Canary Analysis at Netflix with Kayenta**  
https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69

この自動分析をGKEで試そうとしていたが、Kubenetes(k8s)コンテナー(Stackdriver)からメトリクスが取れていなかった。
理由がわかったのでメモしておく。


<!--more-->

**2018/07/06追記**  
なお、この記事はSpinnaker v1.8.0を利用したときの記事である。  
すでにこの問題はPRで対応されており、tagを見る限り1.9.0では改善される見込みだ。

- feat(stackdriver): Add support for legacy gke_container resource type.
  - https://github.com/spinnaker/kayenta/pull/337

コメントで教えていただきありがとうございました。

# TL;DR
- Spinnaker1.8.0がサポートしているメトリクスのResource Typeに`gke_container`は含まれていない
- そのため、k8sを`1.10`系にしてStackdriver k8s MonitoringのBeta版を有効にする必要がある。

```bash
GCP_PROJECT=$(gcloud info --format='value(config.project)')
CLUSTER_NAME=cluster-1
CLUSTER_VERSION=1.10.4-gke.2
ZONE=us-central1-a
CLOUDSDK_CONTAINER_USE_V1_API=false
CLOUDSDK_API_CLIENT_OVERRIDES_CONTAINER=v1beta1
gcloud beta container clusters create $CLUSTER_NAME \
  --zone=$ZONE \
  --project=$GCP_PROJECT \
  --cluster-version=$CLUSTER_VERSION \
  --enable-stackdriver-kubernetes \
  --enable-legacy-authorization
```

# Kayentaにメトリクスデータが入ってこない
先日GKE上に自動カナリー分析を有効にしたSpinnakerを構築した。

- [GKE上でStackdriverと連携したSpinnaker/Kayenta環境を構築する #spinnaker #kayenta](/2018/06/23/how-to-start-auto-canary-analysis/)

ただ、実際にパイプラインを動かしてもStackdriver上では見えるメトリクスがSpinnaker上からは見えず、ずっと自動カナリーリリースが失敗していた。

# Spinnakerが使うメトリクスのリソースタイプを確認する。

2018/06/27現在のSpinnakerの最新バージョン(1.8.0)でstackdriverの自動カナリー分析の設定を有効にする(`hal config canary google edit --gcs-enabled true   --stackdriver-enabled true`)と、メトリクスの`Resource Type`は以下になっている。

![Resource Types](/2018/06/27_resource-types.png)

- `gce_instance`
- `aws_ec2_instance`
- `gae_app`
- `k8s_container`
- `k8s_pod`
- `k8s_node`
- `global`

ここでStackdriver MonitoringのResource Typeを確認すると、通常のGKEクラスターのResource Typeは`gke_container`になっている。

**Monitored Resource Types**  
https://cloud.google.com/monitoring/api/resources#tag_gke_container

Resource Typeが異なるため、メトリクスを収集することができていなかった。

# k8sクラスターを`1.10`系にしてStackdriver Monitoring Beta版を有効にする
GKE上のk8sクラスターのモニタリングResource Typeを有効にするにはStackdriver MonitoringをBeta版にすれば良い。


**Migrating to Kubernetes Monitoring**  
https://cloud.google.com/monitoring/kubernetes-engine/migration

自分は新しくk8sクラスターを作り直した。

```bash
GCP_PROJECT=$(gcloud info --format='value(config.project)')
CLUSTER_NAME=cluster-1
CLUSTER_VERSION=1.10.4-gke.2
ZONE=us-central1-a
CLOUDSDK_CONTAINER_USE_V1_API=false
CLOUDSDK_API_CLIENT_OVERRIDES_CONTAINER=v1beta1
gcloud beta container clusters create $CLUSTER_NAME \
  --zone=$ZONE \
  --project=$GCP_PROJECT \
  --cluster-version=$CLUSTER_VERSION \
  --enable-stackdriver-kubernetes \
  --enable-legacy-authorization
```

この状態でk8sでコンテナーを起動して、GCP Consoleから[Stackdriver Logging](https://console.cloud.google.com/logs/viewer)を確認する。
`GKE Container`ではなく、`Kubernetes Container`というResourceが選択できるようになったら変更成功だ。

![Exsits k8s Container](/2018/06/27_exists-resource-k8s-container.png)

これでSpinnakerからStackdriver経由でGKE上のメトリクスを参照できるようになった。

![Metrics on Spinnaker](/2018/06/27_metrics-on-spinnaker.png)

# 終わりに
なお、この新しいStackdriver k8s Monitoringは2018/06/27現在まだbeta releaseなのでSLAの対象外になっている。

https://cloud.google.com/monitoring/kubernetes-engine/migration

> This is a Beta release of Stackdriver Kubernetes Monitoring. This feature is not covered by any SLA or deprecation policy and might be subject to backward-incompatible changes.

自分はお試しで触っているのでSLAなど気にしないのだが、本番環境で自動カナリーリリース分析をするにはPrometheus経由とかになるのかな？


# 参考
- [New Stackdriver Monitoring for Kubernetes (Part 1)](https://medium.com/google-cloud/new-stackdriver-monitoring-for-kubernetes-part-1-a296fa164694)
- [Migrating to Kubernetes Monitoring](https://cloud.google.com/monitoring/kubernetes-engine/migration)

# 関連
- [GKE上でStackdriverと連携したSpinnaker/Kayenta環境を構築する #spinnaker #kayenta](/2018/06/23/how-to-start-auto-canary-analysis/)
- [hal config canary google account addすると"Required 'compute.projects.get' permission for..."で403エラー #spinnaker #kayenta](/2018/06/13/403-error-when-add-google-account-for-kayenta/)
- [Spinnakerタグの記事一覧](/categories/spinnaker/)
