+++
title= "[発表資料]Spinnaker入門 #cndjp5"
date= 2018-04-27T10:09:26+09:00
draft = false
slug = ""
categories = ["spinnaker", "presentation","kubernetes"]
tags = ["spinnaker","cndjp","k8s"]
author = "budougumi0617"
twitterImage = "logos/spinnaker.png"
keywords = ["Spinnaker", "Continuous Delivery", "Kubernetes", "k8s"]
+++

Cloud Native Developers JP 第5回勉強会(cndjp5)の発表資料と資料中の参考リンクです。  
https://cnd.connpass.com/event/84310/

<script async class="speakerdeck-embed" data-id="3f26a8a1f1f44cb2ae26c2cda5978edd" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# Spinnaker

**Spinnaker**  
https://www.spinnaker.io


SpinnakerはKubernetesなどに対応した継続的デリバリーツール。  
Blue/Greenやカナリーリリースが簡単にできる

# Quick Start Spinnaker

**Demo/Evaluation Installations**  
https://www.spinnaker.io/setup/quickstart/  
とりあえず試すときはAWS, GCP, Azureで1 click deployが可能。

**Spinnaker Chart**  
https://hub.kubeapps.com/charts/stable/spinnaker  
Helm Chartでもリリースできる。

**Choose your Environment**  
https://www.spinnaker.io/setup/install/environment/  
フル機能でインストールしたいときはHalyardからインストールする。

**Enterpricse-grade Spinnake installed in Kubernetes**  
https://www.armory.io/

フルマネージドのSpinakerもあるらしい。

**Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine**  
https://cloud.google.com/solutions/continuous-delivery-spinnaker-kubernetes-engine  

**GKEでSpinnakerを使ったKubernetesの継続的デリバリーを試してみた**  
https://budougumi0617.github.io/2018/04/24/spinnaker-with-google-kubernetes-engine/  

デモで作ったのはこのサンプル。

# Pipelineの設定について

**Codifying your Spinnaker Pipelines - The Spinnaker Community Blog**  
https://blog.spinnaker.io/codifying-your-spinnaker-pipelines-ea8e9164998f  

Spinnakerのパイプライン設定をコードで編集する流れです。（仕様策定中）

# Spinnake + Kayenta - Automated Canary Analysis

**Automated Canary Analysis at Netflix with Kayenta**  
https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69


**Spinnaker Meetup - Automated Canary Analysis and Deploying to Redhat OpenShift**  
https://www.youtube.com/watch?v=K22lyoopRxk  

先日行われたSpinnaker Meetupの動画。動いている自動カナリー分析

**Deploy like Netflix and Google with Automated Canary Analysis**  
https://cloudonair.withgoogle.com/events/americas?tab=may-1&expand=segment:netflix-canary-netflix-canary  

5/1 25:00(5/2 午前1時)にCloud On Airもある。

**Set up canary support**  
https://www.spinnaker.io/setup/canary/  
この通りにやればKayentaを有効化できそうだけどまだ成功していない。

# Community
**Slack**  
https://join.spinnaker.io/  

**Stack Overflow**  
https://stackoverflow.com/questions/tagged/spinnaker  
Slackでエラーメッセージを検索するとだいたい出てくる。


