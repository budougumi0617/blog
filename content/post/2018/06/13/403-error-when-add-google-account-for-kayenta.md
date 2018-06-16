+++
title= "hal config canary google account addすると\"Required 'compute.projects.get' permission for...\"で403エラー #spinnaker #kayenta"
date= 2018-06-13T09:13:51+09:00
draft = false
slug = ""
categories = ["spinnaker", "kubernetes", "gcp"]
tags = ["spinnaker","gcp","kayenta", "k8s"]
author = "budougumi0617"
+++

SpinnakerはKubenetesクラスターへの継続的デリバリーを可能とするOSSだ。  
Kayenataを有効化することで、カナリーリリースの自動スコアリングも可能とする。

**Introducing Kayenta: An open automated canary analysis tool from Google and Netflix**  
https://cloudplatform.googleblog.com/2018/04/introducing-Kayenta-an-open-automated-canary-analysis-tool-from-Google-and-Netflix.html

しかし2018/06/13現在、少なくともv1.7.0ではCanary Analytics(Kayenta)はデフォルトで無効化されている。

今回はSpinnakerのKayentaを有効化する途中、GCP用の設定をしようとして以下のエラーが出たときのメモ。

```bash
budougumi0617@budougumi0617-halyard:~$ hal config canary google account add my-google-account \
>   --project budougumi0617-spinnaker6 \
>   --json-path ~/.gcp/gcp.json
+ Get current deployment
  Success
+ Get all canary settings
  Success
- Add the my-google-account canary account to google service integration
  Failure
Problems in default.canary:
! ERROR Failed to load project "budougumi0617-spinnaker6": 403
  Forbidden
{
  "code" : 403,
  "errors" : [ {
    "domain" : "global",
"message" : "Required 'compute.projects.get' permission for
  'projects/budougumi0617-spinnaker6'",
    "reason" : "forbidden"
  } ],
"message" : "Required 'compute.projects.get' permission for
  'projects/budougumi0617-spinnaker6'"
}.

- Failed to add canary account my-google-account for service
  integration Google.
```

# TL;DR
- [Set up canary support](https://www.spinnaker.io/setup/canary/)のQuick start通りにやると403エラーが出る
- Service Account(SA)に`roles/compute.viewer`ロールを付与すれば解決できる
- 似たような権限エラーのときは[IAMロール一覧](https://cloud.google.com/compute/docs/access/iam)で必要なロールを探してSAに付与する


# 前提条件
Google Kubernetes Engine(GKE)上にv1.7.0のSpinnakerを一度起動している。

Spinnaker自体は以下のquickstart通りにhalyardインスタンスを構成する方法で起動した。

**Try out Halyard on GKE**  
https://www.spinnaker.io/setup/quickstart/halyard-gke/

Spinnakerを起動できたら、以下の手順に則ってKayentaの設定を有効化する。

**Set up canary support**  
https://www.spinnaker.io/setup/canary/

しかし、Stack Driverなどのログ閲覧用のアカウントを作成する途中以下のようなエラーが出てしまった。

```bash
budougumi0617@budougumi0617-halyard:~$ hal config canary google account add my-google-account \
>   --project budougumi0617-spinnaker6 \
>   --json-path ~/.gcp/gcp.json
+ Get current deployment
  Success
+ Get all canary settings
  Success
- Add the my-google-account canary account to google service integration
  Failure
Problems in default.canary:
! ERROR Failed to load project "budougumi0617-spinnaker6": 403
  Forbidden
{
  "code" : 403,
  "errors" : [ {
    "domain" : "global",
"message" : "Required 'compute.projects.get' permission for
  'projects/budougumi0617-spinnaker6'",
    "reason" : "forbidden"
  } ],
"message" : "Required 'compute.projects.get' permission for
  'projects/budougumi0617-spinnaker6'"
}.

- Failed to add canary account my-google-account for service
  integration Google.
```


# 使うService Account事前に`roles/compute.viewer`ロールを付与しておく
エラーは(`--json-path`オプションで参照している)Service Account(SA)に必要なパーミッションを付与していないのが原因だった。
[Try out Halyard on GKE](https://www.spinnaker.io/setup/quickstart/halyard-gke/)の通りにHalyardサーバーを構成している場合、
上記のコマンドで指定しているJSONのSAは`gcs-service-account`という名前になっているはずなので、それにIAMのロールを付与する。

`compute.projects.get`パーミッションを持つ最小権限のロールを以下のページから探す。

**Compute Engine IAM Roles**  
https://cloud.google.com/compute/docs/access/iam

そうすると、どうやら[roles/compute.viewer](https://cloud.google.com/compute/docs/access/iam#compute_viewer_role)が最小権限のロールなので、これを利用することにする。

自分はHalyardサーバにIAMの編集権限をつけていなかったのでローカルPCで以下の操作を行った。

```bash
GCS_SA=gcs-service-account
GCS_SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:$GCS_SA" \
    --format='value(email)')

gcloud projects add-iam-policy-binding budougumi0617-spinnaker6 \
    --member serviceAccount:$GCS_SA_EMAIL \
    --role roles/compute.viewer
```

上記の操作を行ったあと、再び`hal`コマンドを実行すると以下のようにCanary(Kayenta)用のGCPアカウントを作成できた。

```bash
budougumi0617@budougumi0617-halyard:~$ hal config canary google account add my-google-account \
  --project budougumi0617-spinnaker6  \
  --json-path ~/.gcp/gcp.json
+ Get current deployment
  Success
+ Get all canary settings
  Success
+ Add the my-google-account canary account to google service integration
  Success
+ Successfully added canary account my-google-account for service
  integration Google.
```

ちなみに`my-google-account`というのはGCP上の設定とは関係ないので、Spinnaker上で自分が区別できる名前ならば何でも良い（はず）。

# 終わりに
自分はSREやインフラエンジニアでないので、クラウドサービスのIAMやKubernetesのRBACらへんをいまいちわかっていなかったが、GCPのIAMとService Accountの概念をすこしわかってきた気がする。  
とはいえ、技術調査のときは使うService Accountに最初からフルコントロールをつけた状態でやったほうが無駄に消耗しなくてよさそうだ。


# 関連
- [Spinnakerの始め方いろいろ](/2018/04/22/how-to-start-spinnaker/)
- [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)
- [GKEでSpinnakerを使ったKubernetesの継続的デリバリーを試してみた](/2018/04/24/spinnaker-with-google-kubernetes-engine/)
- [[Spinnaker]GKE上でhal deploy applyするとkayenta failed http://localhost:8090/spectator/metrics with <urlopen error [Errno 111] Connection refused> #spinnaker #kayenta](/2018/06/16/spinnaker-failing-with-stackdrivermetricsservice/)

