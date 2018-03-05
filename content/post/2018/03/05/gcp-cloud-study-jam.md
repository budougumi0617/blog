+++
title= "GCPハンズオン( #CloudStudyJam )に参加した"
date= 2018-03-05T13:11:11+09:00
draft = false
slug = ""
categories = ["report", "gcp"]
tags = ["gcp"]
author = "budougumi0617"
+++


今年から自己学習でGCPを触り始めている。
ちょうど土曜日にGCPのハンズオンに開催されたので、参加してきた。以下ハンズオン内容と自分メモ。

|||
|---|---|
|URL|https://www.meetup.com/ja-JP/gdg-tokyo-jp/events/248338420/|
|会場|DMM New Office|
|日時|2018/03/03(土) 13:00 〜 17:00|
|会費|1,000円|

# TL;DR
- GCP初めて触る人向けハンズオン
- QwikLabs（e-ラーニングサイト）のチュートリアルを使ってGCPの基本サービスを学ぶ
  - [仮想マシンの作成](https://google.qwiklabs.com/focuses/5318)
  - [Cloud Launcher を使用したサービスの プロビジョニング](https://google.qwiklabs.com/focuses/5322)
  - [永続ディスクの作成](https://google.qwiklabs.com/focuses/6988)
  - [Weather Data in BigQuery](https://google.qwiklabs.com/focuses/5689)
- 当日以内（24時間以内なのかも）にEssentialsの残りの内容の学習を終えるとQwikLabが1ヶ月無料
  - [GCP Essentials](https://google.qwiklabs.com/quests/23)
- BiqQueryでお金を溶かさないTips
- [Kubernetes in the Google Cloud](https://google.qwiklabs.com/quests/29)


# ハンズオン内容

[QwikLabs](https://qwiklabs.com/?locale=ja)はAWS、GCPの教材があるe-ラーニングサイト。  
チュートリアルごとに使い捨てのGCPアカウントなどを作成して学習することができる。そこそこ翻訳済みのチュートリアルも多い。  
今回のハンズオンでは[GCP Essentials](https://google.qwiklabs.com/quests/23)の中から３つ、それとは別Bigqueryのチュートリアルの合計4つを行った。

- [仮想マシンの作成](https://google.qwiklabs.com/focuses/5318)
- [Cloud Launcher を使用したサービスの プロビジョニング](https://google.qwiklabs.com/focuses/5322)
- [永続ディスクの作成](https://google.qwiklabs.com/focuses/6988)
- [Weather Data in BigQuery](https://google.qwiklabs.com/focuses/5689)

# BigQueryの豆知識
BigQueryはクエリを走らせたときの検索量によって課金されるサービス。  
いくつか口頭でお金を溶かさない方法を教えてもらったのでメモしておく。

## Web UIでSELECT *していると警告がでる
BiqQueryはWeb画面でSQL文を入力し、クエリを実行することができる。このときカラムの量でも料金が変わる。  
もし`SELECT *`なんて書いていた場合、警告を受ける。

## Web UIでクエリを実行するまえに料金の目安がわかる
[Using the query validator](https://cloud.google.com/bigquery/docs/best-practices-costs?hl=ja#price_your_queries_before_running_them)


Web UIでクエリを書くと、オートでSQLのvalidationをしてくれる。  
この時、クエリ実行時に確認するデータサイズを見積もってくれるため、ここからクエリ実行時にかかる料金を見積もることができる。  
AWSのAthenaはこういうのなかった気がするのでなかなかありがたい！

## Maximum Bytes Billedオプションを設定しておく
[Limit query costs by restricting the number of bytes billed](https://cloud.google.com/bigquery/docs/best-practices-costs#limit_query_costs_by_restricting_the_number_of_bytes_billed)

(日本語には翻訳されていないらしく、英語のページにしか説明がない)

定期的にクエリを実行していると、もしかしたらあるタイミングでデータ総量が変化（バズったとか）し、いつもと違う金額になるかもしれない。  
そんなときでも`Maximum Bytes Billed`オプションを設定しておくとそれ以上の料金になるクエリが実行されることがなくなるので安心。


# ハンズオンをうけてみて

## GCPはGoogle Cloud Shell上でなんでも出来る
チュートリアルはだいたいコマンドをコピペしながら進めていくのだが、  
GCPはクラウド上でCloud Shellを起動するだけなので、Chromeさえインストールされていれば問題なかった。  
当然なれてくればローカルでもやれるようにしておきたいが、こういう「とりあえず触ってみる」というときに何も環境構築せずに始められるのはとても良いと思う。

## QwikLabsを独りで始めるのは大変かも。。。？
今回はハンズオンでチューターの方がすぐに助けてくれる状態だったので、
最後までいけたが、進捗ステータスがちゃんと反映されなかったり、初回のアクティベーションで詰まっている方も何人かいたようだ。  
（自分も進捗がうまく反映されてない気がして一度やり直したりした。）  
QwikLabsをキャンペーンコード無しでやろうとするとそこそこお金がかかるみたい(GCP Essentialsだけでも1,800円くらい？)。  
GCPの公式チュートリアルもかなり量がある・フォロー範囲が広いので、GCPの1年間無料登録をして、GCP公式のチュートリアルを触るほうがよいかもしれない。

https://cloud.google.com/getting-started/?hl=ja

# 最後に
参加費1,000円だったが、懇親会で料理を頂いたりかなり得してしまった。GDG・GCPUGのみなさんありがとうございました。  
ハンズオン後自宅でGCP Essentialsの残りのチュートリアルも全部行なったので、QwikLabsが1ヶ月無料利用に。

[https://google.qwiklabs.com/public_profiles/](https://google.qwiklabs.com/public_profiles/0e57dfba-8085-491f-8932-7eb3da012190)

無料枠で早速Go/AppEngineのチュートリアルは実施した。dev_appserver.pyを使うとGoのアプリもホットリロードしながら開発が出来る。すごい。

- [App Engine: Qwik Start - Go](https://google.qwiklabs.com/focuses/10416)

この後はkubernetesのチュートリアルを行うつもり。

- [Kubernetes in the Google Cloud](https://google.qwiklabs.com/quests/29)

kubernetesの方は総額54クレジット(1クレジット = 1ドル)なので、かなりお得に勉強できる。せっかく頂いた機会なのでしっかり勉強するぞ！


