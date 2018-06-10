+++
title= "コンテナ技術の学習するなら圧倒的にKatacodaがおすすめ #k8s #kubernetes"
date= 2018-06-10T16:54:38+09:00
draft = false
slug = ""
categories = ["Kubernetes", "docker"]
tags = ["k8s", "docker"]
author = "budougumi0617"
+++

https://www.katacoda.com/

![lesson-image](/2018/06/katacoda-top.png)

Katacodaはコンテナ技術に強い無料の学習サイトだ。  
ブラウザさえあれば簡単にKubernetesなどを動かせるので、とりあえず触ってみたい人におすすめしたい。


# TL;DR
- katacodaは無料で利用できる学習サイト
- コンテナ周辺技術をブラウザオンリーで動かしながら学習することができる
- とりあえず触ってみたい人におすすめ
- チュートリアルのUIが良くてタイポも気にせずに学習できる
- 各言語、ツールのPlaygroudも豊富
- 技術背景や概論については別でキャッチアップが必要

# Katacodaは無料で利用できる学習サイト
Katacodaはブラウザ上で利用できるeラーニングサイトだ。
利用は無料で自分で新しい学習コースを作成することもできる。

https://www.katacoda.com/

登録をしなくても利用できるが、進捗などを記録したほうがやる気が出る人は無料登録をしておくとよい。

![lesson-image](/2018/06/katacoda-progress.png)

# コンテナ周辺技術をブラウザオンリーで動かしながら学習することができる
Katacodaに登録してあるレッスンの一覧はこちらから確認することができる

https://www.katacoda.com/learn

![lesson-image](/2018/06/katacoda-lesson-list.png)

DockerやkubernetesのレッスンだけでなくIstioやTerraformなどのレッスンもある。
また、GitやJava、Goなどのレッスンも存在する。
どれもすべてWebブラウザ内で完結するため、ローカルに事前に環境を用意する必要はない。  
例えばKubernetesについては2018/06/10時点で17レッスン存在する。
NetworkやSecretsなどの基本から、Helmなどの周辺ツールのレッスンもある。

- Launch Single Node Kubernetes Cluster
  - https://www.katacoda.com/courses/kubernetes/launch-single-node-cluster
- Networking Introduction
  - https://www.katacoda.com/courses/kubernetes/networking-introduction
- Use Kubernetes to manage Secrets
  - https://www.katacoda.com/courses/kubernetes/managing-secrets
- Helm Package Manager
  - https://www.katacoda.com/courses/kubernetes/helm-package-manager

# チュートリアルのUIが良くてタイポも気にせずに学習できる
各レッスンはブラウザ場でエディタやターミナルを操作しながら進める。

![lesson-image](/2018/06/katacoda-contents.png)

レッスン中のコマンドやファイルの内容はクリックすると自動で右のターミナルやエディタにコピーされる。  
なので「悩んだ挙げ句結局タイポしていた」みたいなことで時間を無駄にすることもない。  
もちろん自分で自由にコマンドを入力することも可能なので、レッスン内容にないコマンドを実行して気になる情報を確認したりすることも可能だ。

# 各言語、ツールのPlaygroudも豊富
https://www.katacoda.com/learn#playgrounds

チュートリアルだけでなく、オンラインエディタ/オンライン環境上でちょっと何か試したいときのPlaygroudも豊富に用意されている。

![Playgrounds](/2018/06/katacoda-playgrounds.png)

ざっと書くと以下のPlaygroundsがある。

- Docker
- Kubernetes
- Tensorflow
- Golnag
- mono/C#
- Scala
- R language
- etc


# 技術背景や概論については別でキャッチアップが必要
新しい技術を事前準備なしでブラウザだけで学習できる気軽さがKatacodaの魅力だが、Katacodaだけでなく技術書や公式サイトでの学習は必要だろう。
Katacodaのレッスンは手順的な内容が多いので、
「なぜこの手段を使うのか？」「なぜこれが必要になるのか？」（たとえばなぜLiveness HealthcheckとReadiness Healthcheckの違いは何か？）
という知識は別の場所で学習する必要がありそうだ。  
また、各コンテナ技術を業務で使う際はほぼクラウドベンダーのサービス上で使うことになるだろう。
特定のクラウドベンダー上で使う際の手順（IAMの設定の仕方など）はないので、Katacodaの学習が終わったあとは
自分が使うクラウドサービスのチュートリアルにトライしてみれば良いと思う。

- GCP | Kubernetes Engine ドキュメント
  - https://cloud.google.com/kubernetes-engine/docs/
- Kubernetes を伴う Azure Container Service のドキュメント
  - https://docs.microsoft.com/ja-jp/azure/container-service/kubernetes/
- AWS Workshop for Kubernetes
  - https://github.com/aws-samples/aws-workshop-for-kubernetes


# 終わりに
「なんか難しそう」と思う技術もまず触ってみればそこまで難しくないことも多い。  
環境構築でつまずいて諦めてしまうよりもeラーニングサイトなどを利用してでまず軽く触ってみるとよいと思う。

