+++
title= "[Spinnaker]GKE上でhal deploy applyするとkayenta failed http://localhost:8090/spectator/metrics with <urlopen error [Errno 111] Connection refused> #spinnaker #kayenta"
date= 2018-06-16T05:50:45+09:00
draft = false
slug = ""
categories = ["spinnaker"]
tags = ["spinnaker", "canary", "kayenta", "k8s"]
author = "budougumi0617"
+++
Spinnakerで自動リリース分析をするための環境構築をしている。

**Try out Halyard on GKE**  
https://www.spinnaker.io/setup/quickstart/halyard-gke/

**Set up canary support**  
https://www.spinnaker.io/setup/canary/

Spinnakerの設定を変更して`hal deploy apply`したらそのままレスポンスが帰ってこなくなった。
表題のエラー内容をloggingで見つけるまでだいぶ悩んでしまったのでメモ。

# TL;DR
- Stackdriver周りの設定を有効にしてSpinnakerを再起動しようとしたらコマンドが終わらなくなった
- Stackdriver loggingでGKEコンテナのログを調べた。
- `StackdriverMetricsService`への書き込みエラーを確認
- `SIGKIL`後、`hal deploy clean`でSpinnakerのコンテナを破棄
- 利用しているサービスアカウントにStackdriverとモニタリングの編集権限をつけた
- `hal deploy apply`で再起動することを確認

# `hal deploy apply`が終わらない


Kayentaを有効にしたが、Stackdriverを有効にしていなかった。
そのため、Stackdriver有効後Spinnakerを再起動しようと`hal deploy apply`したところ、エラーレスポンスもなくずっとコマンドが終わらなく鳴ってしまった

# とりあえずGKEのコンテナのログをみる。
何か様子がおかしくなったらまずstackdriver loggingでGKEコンテナのログを見てみる。

https://console.cloud.google.com/logs

ログを見るときは「リソース」がGKEコンテナになっていることを確認すること。  
自分は「Kubernetesクラスタ」のログを見ていて（別の場所を見ていて）「エラーログがない」と勘違いしていた。

ログを確認してみると、以下のようなエラーが繰り返し出続けていた。

```
2018-06-16 03:31:44.000 JST
18:31:44 Caught <HttpError 403 when requesting https://monitoring.googleapis.com/v3/projects/PROJECT_ID/timeSeries?alt=json returned "User is not authorized to access the project monitoring records: Requires permission monitoring.timeSeries.create">
```
```
2018-06-16 03:35:59.000 JST
18:35:59 Caught <HttpError 400 when requesting https://monitoring.googleapis.com/v3/projects/PROJECT_ID/timeSeries?alt=json returned "Field timeSeries[106].metricKind had an invalid value of "GAUGE": Metric kind for metric custom.googleapis.com/spinnaker/clouddriver/redis.command.payloadSize.eval must be CUMULATIVE.">
```

エラーを見ればなんのことはない、結局サービスアカウントの権限不足だった。  
ログをもう少し漁ると、以下のようなログも見つかるはずだ。

```
2018-06-16 04:50:11.000 JST
19:50:11 Using Stackdriver Credentials from "/home/USER/.hal/default/staging/dependencies/926556552-gcp.json"
```

# IAMロールを修正してSpinnakerを再起動する

上記のjsonの中をみると紐付いているサービスアカウントのメールアドレスがわかるので、

```
cat .hal/default/staging/dependencies/926556552-gcp.json
{
  "type": "service_account",
  "project_id": "PROJECT_ID",
  "private_key_id": "9...",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMII...
  "client_email": "gcs-service-account@???.iam.gserviceaccount.com",
  ....
}
```

上記のサービスアカウントにモニタリング周りのIAMロールを追加で付与することでエラーは解決した。

- Stackdriver Profiler エージェント
- モニタリング編集者
- Stackdriver リソース メンテナンス時間枠の編集者


（正直つけすぎたかもしれない）

# 終わりに
相変わらずIAM周りの設定でドハマりしてしまった。  
普段自分はまったくインフラを直接触らないので、もう技術調査をしているときに使うサービスアカウントは最初オーナー権限にしてしまったほうが良さそうだ。


# 関連
- [Continuous Delivery With Spinnakerでマイクロサービスの継続的デリバリーの課題と解決手法を学ぶ](/2018/06/14/review-continuous-delivery-with-spinnaker/)
- [hal config canary google account addすると"Required 'compute.projects.get' permission for..."で403エラー #spinnaker #kayenta](/2018/06/13/403-error-when-add-google-account-for-kayenta/)
- [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)
- [Spinnakerの始め方いろいろ](/2018/04/24/spinnaker-with-google-kubernetes-engine/)

