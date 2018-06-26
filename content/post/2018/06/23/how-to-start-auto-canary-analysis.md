---
title: "GKE上でStackdriverと連携したSpinnaker/Kayenta環境を構築する #spinnaker #kayenta"
date: 2018-06-23T18:28:42+09:00
draft: false
toc: true
categories:
- spinnaker
- kubernetes
tags:
- spinnaker
- kayenta
- k8s
---

SpinnakerとKayentaを組み合わせ自動カナリー分析を行うための環境構築を行う。
今回の構築ではGKE上にSpinnakerを構築し、Stackdriverから分析用のログを取得する。


2018/06/27追記  
なお利用するkubernetes(k8s)クラスターは`v1.10`系で構築し、ベータ版のモニタリングを有効にしておくこと。  

- [SpinnakerでGKEの自動カナリー分析をするときはStackdriver k8s Monitoring V2 BETAを使う #spinnaker #kayenta](/2018/06/27/gke-must-use-beta-monitoring-for-kayenta)

# TL;DR
- "Try out Halyard on GKE"の通りにHalyardインスタンスを作成する
  - https://www.spinnaker.io/setup/quickstart/halyard-gke/
- サービスアカウントにStackdriver周りの権限を付与する
- "Set up canary support"の通りにGCP関連の設定をONにして`hal deploy apply`する
  - https://www.spinnaker.io/setup/canary/

#  "Try out Halyard on GKE"の通りにHalyardインスタンスを作成する
Kayentaを有効にするためには、Halyard経由で操作可能なSpinnakerを用意する必要がある。  
GKEとHalyardを使ったSpinnakerの起動は以下のページの通りにすればよい。  
`hal deploy apply`する手前まで実施する。

**Try out Halyard on GKE**  
https://www.spinnaker.io/setup/quickstart/halyard-gke/

# サービスアカウントにStackdriver周りの権限を付与する
一通りの準備が終わったあとはモニタリング周りの準備をしておく。当然ながらStackdriverは有効にしておくこと。

サービスアカウントにStackdriverへアクセス権限を付与しておく。  
`$GCS_SA_EMAIL`などは`Halyard`の準備をするときに取得したサービスアカウントのEメールアドレスだ。

```bash
gcloud projects add-iam-policy-binding $GCP_PROJECT \
    --role roles/logging.admin \
    --member serviceAccount:$GCS_SA_EMAIL

gcloud projects add-iam-policy-binding $GCP_PROJECT \
    --role roles/monitoring.admin \
    --member serviceAccount:$GCS_SA_EMAIL

gcloud projects add-iam-policy-binding $GCP_PROJECT \
    --role roles/compute.viewer \
    --member serviceAccount:$GCS_SA_EMAIL
```

`roles/compute.viewer`ロールまで使うのは`compute.projects.get`権限が必要なため。

# "Set up canary support"の通りにGCP関連の設定をONにして`hal deploy apply`する
上記が終わった後は、GCP Storageを開き、バケットを作成しておく。  
https://console.cloud.google.com/storage/browser

```
MY_SPINNAKER_BUCKET=budougumi0617-spinnaker-bucket
```

その後は以下の手順のGCP関連の部分に則ってcanary系のコマンドを実行していけばよい。

**Set up canary support**  
https://www.spinnaker.io/setup/canary/

```bash
hal config canary enable
hal config canary google enable
hal config canary google account add my-google-account \
  --project $GCP_PROJECT \
  --json-path $GCS_SA_DEST \
  --bucket $MY_SPINNAKER_BUCKET
hal config canary google edit --gcs-enabled true \
  --stackdriver-enabled true
hal config canary edit --default-metrics-store stackdriver
hal deploy apply
```

`hal deploy connect`コマンドでSpinnakerにつなぎ、`Create Application`でアプリケーションの設定を作る。  
`CONFIG`を開いて、`Features`に`Canary`が表示されていれば自動カナリー分析をする準備が完了している。

![CanarySetting](/2018/06/canary-setting.png)

# 終わりに
今回はそのままカナリー分析を行ってみるところもまで記事にするつもりだったが、実はこの先で詰まってしまっている。  
一通りのパイプラインまで作ったのだが、デプロイ後のログが取得できていない（Stackdriverから設定は読み込めているので権限で弾かれているわけではないはずなのだが。。。）。  
そちらが解決したら実際に自動カナリー分析をする記事も書く。

# 関連
 - [hal config canary google account addすると"Required 'compute.projects.get' permission for..."で403エラー #spinnaker #kayenta](/2018/06/13/403-error-when-add-google-account-for-kayenta/)

