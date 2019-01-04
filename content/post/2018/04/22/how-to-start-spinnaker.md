+++
title= "Spinnakerの始め方いろいろ"
date= 2018-04-22T22:47:22+09:00
draft = false
slug = ""
categories = ["Spinnaker"]
tags = ["Spinnaker", "CD",]
author = "budougumi0617"
twitterImage = "logos/spinnaker.png"
keywords = ["Spinnaker", "Continuous Delivery", "Kubernetes", "k8s"]
+++



Spinnakerはマルチクラウドに対応したOSSの継続的デリバリーツール。

**Spinnaker**  
https://www.spinnaker.io

**SpinnakerによるContinuous Delivery**  
http://tech.mercari.com/entry/2017/08/21/092743

Spinnakerを試してみたいと思うにあたって、まずSpinnaker自体の環境構築方法を調べてみた。


# TL;DR
- とりあえず試すときはAWS, GCP, AzureでQuick Start
  - https://www.spinnaker.io/setup/quickstart/
- それ以外でとりあえず試すときはHelm Charts
  - https://hub.kubeapps.com/charts/stable/spinnaker
- ちゃんとインストールしたいときはHalyardから
  - https://www.spinnaker.io/setup/install/environment/
- ローカルで動かすのはどうだろう？
- 実際にパイプラインを動かしてみるならばGCPがおすすめ

# とりあえず試すときはAWS, GCP, AzureでQuick Start
https://www.spinnaker.io/setup/quickstart/

ひとまず動かすだけでよいならば、それぞれのクラウドプロパイダーでボタン一つで起動できるクイックスタートが用意されている。

**AWS | Spinnaker on AWS**  
https://aws.amazon.com/jp/quickstart/architecture/spinnaker/

**GCP | Cloud Lancher**  
https://console.cloud.google.com/launcher/details/click-to-deploy-images/spinnaker

**Azure | Quick Start Template**  
https://azure.microsoft.com/ja-jp/resources/templates/?term=Spinnaker

また、Helmのテンプレートもあるので、Helmを使うことでインストールをすることもできる。

**kubeapps.com | Spinnaker**  
https://hub.kubeapps.com/charts/stable/spinnaker

# ちゃんとインストールしたいときはHalyardから
設定などを自分で編集しながら試したいときはDebianへHalyardのインストールから始めるほうがよさそう。

**Install Halyard**  
https://www.spinnaker.io/setup/install/halyard/

HalyardはSpinnakerを管理する`hal`コマンドのこと。
メルカリさんが詳しく書いてくれているのでそちらも参考になりそう。

**次世代Continuous DeliveryプラットフォームであるSpinnakerを体験してみよう！**  
http://tech.mercari.com/entry/2017/12/17/205719

自分もKayentaなどを試したいのでこちらからやる方法を試す。

# ローカルで動かすのはどうだろう？

## それなりのマシンスペックDebian環境が要求されるはず？

**Required Hal Invocations**  
https://www.spinnaker.io/setup/install/environment/#required-hal-invocations-1

> Kubernetes Note: We recommend having at least 4 cores and 8 GiB of RAM free in the cluster you are deploying to.

k8s上で動かすときの注意書きとして上記のクラスター環境が書かれているので、それなりに強めの環境をつくらないと動かなそう？

## 本番を考慮するならばクラウド上で構築しないと試せないこともある

本番環境で使うことを考えたとき、クラウドストレージやクラウドプロパイダーのアクセス権限などとの連携は必須になるはず。
その辺の知見を貯める必要もあることを考えると、ローカルで組むと二度手間になりそうだ。


# 終わりに | 実際にパイプラインを動かしてみるならばGCPがおすすめ
公式ページをチェックしただけの内容になってしまったが、Spinnakerの始め方について調べてみた。  
「インストールしたあと、何ができるのかまで知りたいんだけど」という方もいると思う。  
そういう方はGCPの以下のハンズオンをしてみるのがおすすめ

**Spinnaker と Container Engine を使用した継続的デリバリー パイプライン**  
https://cloud.google.com/solutions/continuous-delivery-spinnaker-container-engine?hl=ja

実際に一連のGCPのサービスに連携させた状態のSpinnakerを使ってパイプラインを動かしてみることができる。


