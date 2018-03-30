+++
title= "Kubernetes Network Deep Dive!（Istioもあるよ） cndjp第四回参加メモ #cndjp4"
date= 2018-04-01T11:19:22+09:00
draft = false
slug = ""
categories = ["report","k8s"]
tags = ["k8s","cndjp", "istio"]
author = "budougumi0617"
+++

オラクル[@hhiroshell](https://twitter.com/hhiroshell)さん主催のcndjp。
今回はKubernetes(k8s)のネットワーク構成について、そしてIstioの概要とそれを利用したサービスメッシュ構成のハンズオンを行った。

|||
|---|---|
|URL|https://cnd.connpass.com/event/79999/|
|会場|オラクル青山センター|
|日時|2018/03/28(水) 18:30 〜 20:30 |
|資料1|[Kubernetes Network Deep Dive!](https://speakerdeck.com/hhiroshell/kubernetes-network-deep-dive/)|
|資料2|[Istio入門](https://speakerdeck.com/hhiroshell/istioru-men)|
|資料3|[ハンズオン資料](https://github.com/oracle-japan/cndjp4)|
|ハッシュタグ| [cndjp4](https://twitter.com/search?q=%23cndjp4)|


# TL;DR
- k8sのクラスターネットワークは３つのレイヤに分けて考えるとわかりやすい
  - Pod Network
  - Service Network
  - クラスタ外への公開
- Istioを利用してサービスメッシュ構成を作成する
  - サービスメッシュの概要とそのメリット
  - Istio概要

# k8sクラスター・ネットワークの全体像
前半はk8sのクラスターネットワークについての勉強だった。

## Pod Network
コンテナの実行環境に対してk8sは以下のネットワーク構成を必要とする
（このネットワーク構成はk8s自体の機能ではない）

- 全てのコンテナがNATなしで他のコンテナと通信可能
- 全てのノードがNATなしで他のコンテナと通信可能
- コンテナが自分のIPを参照するとき、そのIPは他のコンテナから見たときと同じでないと行けない。

同じPod内のコンテナは同じIPになるので、Pod内のコンテナはそれぞれ異なるPort番号が割り当てられる。
他のコンテナとPort番号がバッティングした場合はコンテナを立ち上げることができない。

上記の条件の実現方式はFlannelやAdvanced Routing(GCE)などが提供されている。

https://kubernetes.io/docs/concepts/cluster-administration/networking/


## Service Netwwork
複数Podに対するルーティングやロードバランシングを実現するのがService Network。
KubernetesオブジェクトでいうとServiceオブジェクトが該当する。
k8sのServiceはLabelに応じてルーティングするPodを識別するが、Nodeのiptables、kube-proxyによって成り立っている。

- PodとServiceの間には、EndpointというLabel Selectorに当てはまるPodのIP、Portの対を保持するリソースが存在する。
- kube-proxyがServiceとEndpointの設定をWatchして各Nodeのiptablesを更新する

- [GKE/Kubernetes の Service はどう動いているのか](https://speakerdeck.com/apstndb/kubernetes-false-service-hatoudong-iteirufalseka)

## クラスター外への公開
k8sクラスターをクラスター外部に公開するときはいくつか方法がある

- ServiceのNodePortタイプを利用する
- ServiceのLoadBalancerタイプを利用する
- Ingress(beta)を利用する

外部からの通信をうけたk8sのクラスターはiptablesによって接続先を変更し、Pod Networkで後の通信を処理する。

### ServiceのNodePort
単純に対象のPodを外部に公開する口を作成する。

[Service|NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)


### ServiceのLoadBalancer
クラウドプロバイダが提供するLBの自動構成を利用することができる。

GCP, AWS, Azure, OpenStackで利用する際のYAMLの書き方は以下の公式ページに書いてある。
(AWSだけやけに詳しい)

[Service|LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer)

[勉強会の資料(P46)](https://speakerdeck.com/hhiroshell/kubernetes-network-deep-dive/)には「Serviceオブジェクト以上のことはLBに設定することはできない（(OSIの？)L3-L4相当)と書いてあるが、k8sの公式を見るとSSLの設定やアクセスログの設定もできそう

[SSL support on AWS](https://kubernetes.io/docs/concepts/services-networking/service/#ssl-support-on-aws)
[ELB Access Logs on AWS](https://kubernetes.io/docs/concepts/services-networking/service/#elb-access-logs-on-aws)

### Ingress(beta)を利用する
IngressはL7 LoadBalancing を行なうリソース。SSL/TLS終端なども利用することができる。
Ingressオブジェクトに構成情報を記載してIngress Controllerによってプロビジョニングを行う

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
[WHAT IS LAYER 7 LOAD BALANCING?](https://www.nginx.com/resources/glossary/layer-7-load-balancing/)


## ハンズオン
https://github.com/oracle-japan/cndjp4/blob/master/docs/handson1.md

ハンズオンでは作成したServiceオブジェクトを作成したあとEndpointの設定を確認し、`NGINX Ingress Controller`を利用したサービスの外部公開などを試した。

# Istio入門
後半はIstioについて紹介していただたいた。


## サービスメッシュの概要とそのメリット
まずIstioが提供するサービスメッシュの機能とはなにか。
サービスメッシュとはマイクロサービス同士を接続するときに利用するミドルウェア。RESTにおけるmiddlewareパターン、gRPCにおけるInterceptorパターンをマイクロサービス間で実現すること。
セキュリティ、ロードバランシング、ロギングなど、分散システム間に必要だがアプリケーションの本来の機能とは関係ない機能を適用することができる。


## Istioとは

https://istio.io/

Istioはサービスメッシュの機能を実装するオープンソース。意外とCNCFのプロダクトではない(内部で利用しているEnvoyはCNCFのプロダクト）。
サービスメッシュを提供するOSSとしては他に[Linkerd](https://linkerd.io/)などもある。

### アーキテクチャ概要

https://istio.io/docs/concepts/what-is-istio/overview.html#architecture

Istioは主に4種類のコンポーネントから構成される。

- Envoy
  - 各Podにサイドカーパターンで注入される。これがPod間のトラフィックの仲介を行う。
- Mixer
  - Envoyに適用するポリシーを提供しトラフィックの収集を行う
- Pilot
  - Envoyを通してA/Bテストやカナリーデプロイメントやタイムアウト、リトライ回数などの高度なトラフィック管理を行う。
- Istio-Auth
  - マイクロサービス間で利用する認証のための各種情報を提供する


Istioを使うことでサービス間の障害に対する対応やカナリーデプロイメントが簡単に適用できる

- リトライパターン
- サーキットブレーカーパターン
- バルクヘッドパターン
- カナリーデプロイメント

## ハンズオン

https://github.com/oracle-japan/cndjp4/blob/master/docs/handson2.md

ハンズオンではIstio管理下にバージョンが異なるEnvoyを注入したサンプルアプリのPodを起動し、Istioによるトラフィック流量制御を行った。

また、公式サイトの以下のハンズオンが良いとのこと

- [トラフィックの管理](https://istio.io/docs/tasks/traffic-management/)
- [ポリシーの適用](https://istio.io/docs/tasks/policy-enforcement/)
- [メトリック、ログ、トレース情報の収集](https://istio.io/docs/tasks/telemetry/)
- [セキュリティ](https://istio.io/docs/tasks/security/)

# 感想
今回はk8sがコマンドの裏で実行しているネットワークの内部構成やサービスメッシュについて学ぶことができた。
内部的にはiptablesを使っていたりと、昔からの技術、自分の知っている技術の上で成り立っていることを知ることが出来て、別にk8sもまったく新しい技術ではなく、既存技術から地続きで出来ているサービスであることを知れてよかった。

Istioはローカルで実行するには最低でも4GB以上のメモリを利用するらしく、学習するにはそれなりのマシンスペックを用意する必要があるとのこと。
認証や障害時の復旧は自分で実装するのはかなり難儀なため、こういったサービスメッシュの機能は積極的に利用していきたい。
そしてOSI参照モデルのレイヤー層を聞いてもすぐにピンと来ない自分がいるのでこういった基礎、ちゃんと復習しないといけないなと思った。
また、買ったまま積読になっている入門Kubernetesを早く読まなきゃという気持ちになった。

# 参考
- [GKE/Kubernetes の Service はどう動いているのか](https://speakerdeck.com/apstndb/kubernetes-false-service-hatoudong-iteirufalseka)
- [Kubernetes|Service|NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)
- [Kubernetes|Service|LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer)
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [WHAT IS LAYER 7 LOAD BALANCING?](https://www.nginx.com/resources/glossary/layer-7-load-balancing/)
- [Istio](https://istio.io/)
- [Istio|Architecture](https://istio.io/docs/concepts/what-is-istio/overview.html#architecture)
- [Linkerd](https://linkerd.io/)

# 関連
- [Cloud Native Developers JP 第1回勉強会 参加メモ #cndjp1](/2017/11/23/cndjp1/)
- [[k8s]Cloud Native Developers JP 第2回勉強会 参加メモ #cndjp2](/2017/12/18/kubernetes-in-production-cndjp2/)
- [[k8s]Cloud Native Developers JP 第3回勉強会 #cndjp3 参加メモ](/2018/02/03/kubernetes-with-availability-cndjp3/)
