+++
title= "[k8s]Cloud Native Developers JP 第3回勉強会 #cndjp3 参加メモ"
date= 2018-02-03T13:31:14+09:00
draft = false
slug = ""
categories = ["report","k8s"]
tags = ["k8s","chaoskube", "kafka"]
author = "budougumi0617"
+++


いつも楽しく学ばせていただいているオラクル早川さん主催のcndjp。
今回は「Kubernetesで実現する高可用性システム」というテーマでKubernetes(k8s)と可用性について、ハンズオンでは実際にk8sとApache Kafkaを使った構成に対して、chaoskubeによるカオスエンジニアリングを行った。

- [Kubernetes × 可用性 -- cndjp第3回勉強会](https://www.slideshare.net/charlier-shoe/kubernetes-cndjp3/charlier-shoe/kubernetes-cndjp3)
- https://github.com/oracle-japan/cndjp3
  - [CNDJP #3 ハンズオンチュートリアル Part 1](https://github.com/oracle-japan/cndjp3/blob/master/handson1.md)
  - [CNDJP #3 ハンズオンチュートリアル Part 2](https://github.com/oracle-japan/cndjp3/blob/master/handson2.md)

|||
|---|---|
|URL|https://cnd.connpass.com/event/75617/|
|会場|オラクル青山センター 13F セミナールーム|
|日時|2018/01/31(水) 18:30 〜 20:30|
|ハッシュタグ| [#cndjp3](https://twitter.com/search?q=%23cndjp3) |
|資料|[Kubernetes × 可用性 -- cndjp第3回勉強会](https://www.slideshare.net/charlier-shoe/kubernetes-cndjp3/charlier-shoe/kubernetes-cndjp3)|

# TL;DR
- マイクロサービスの4層アーキテクチャごとの可用性
- Kafkaを使ったチャットアプリサービスをk8sにデプロイ
- chaoskubeを使ってカオステストを実施


# マイクロサービス4層アーキテクチャ
[Production-Ready Microservices](http://amazon.jp/dp/B01N48GFCQ)の著者である、Susan Fowler氏はマイクロサービスを4層アーキテクチャとして定義している。

[The Four Layers Of Microservice Architecture](https://www.susanjfowler.com/blog/2016/12/18/the-four-layers-of-microservice-architecture)

- マイクロサービス（サービス本体）
- アプリケーションプラットフォーム（ロギング、監視）
- 通信（サービス間の通信）
- ハードウェア

ハードウェア部分をクラウド、サービス間の通信をk8sに任せる。アプリケーションプラットフォーム以上についてもk8sとの組み合わせで高機能・高可用性を実現できる。
高可用性を実現するためには、レイヤー毎に単一障害点（SPOF）を考える必要がある。

単一障害点…システム全体に影響を及ぼす障害。認証とかDBとか。

# マイクロサービス層の可用性

## マイクロサービスのトレードオフ
[マイクロサービスのトレードオフ](http://postd.cc/microservice-trade-offs/)


マイクロサービスを利用することで開発者はモジュールの分割、個別デプロイなど大きなメリットを得るが、一方で無視できないデメリットも多い。その中の一つとして、サービス境界（サービス間の結合部分）に対する可用性・回復性の担保がある。
マイクロサービスの場合、当然モノリシックな構成よりも個々の機能が分裂されているので、SPOFとなり得る部分が多くなる。

マイクロサービスの設計としては、それらに対してヘルスチェックパターン、、サーキットブレーカーパターンなどを適用して回避する。

- [Pattern: Health Check API](http://microservices.io/patterns/observability/health-check-api.html)
- [Pattern: Circuit Breaker](http://microservices.io/patterns/reliability/circuit-breaker.html)
- [回復性のパターン](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/category/resiliency)
  - [正常性エンドポイントの監視パターン](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/health-endpoint-monitoring)
  - [サーキット ブレーカー パターン](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/circuit-breaker)
- [YouTube:Building Fault Tolerant Microservices](https://www.youtube.com/watch?v=pKO33eMwXRs)

他にもバルクヘッドパターンなどがある。


## サービスメッシュを使ったサービス境界に対する障害対策
- [Istio: マイクロサービスのためのサービスメッシュ](https://www.infoq.com/jp/news/2017/06/istio)

モノリシックなサービスと比較してマイクロサービスの個々のサービスにフォーカスした可用性の担保は簡単になる。  
サービス境界に対する可用性の担保はk8sの場合、サービスメッシュを利用すると簡単に導入できる。  
サービスメッシュとはサービス間のミドルウェアで、モノリシックなサービスだとnginxがやっているような機能をネットワーク間の処理をいい感じにやってくれるツールの総称。IstioやLinkerdなどがある。（これに関しては次回のcndjpで取り上げていただけるらしい）

# ハンズオン1: Apache Kafkaを利用した簡易チャットアプリをk8sにデプロイする

- [CNDJP #3 ハンズオンチュートリアル Part 1](https://github.com/oracle-japan/cndjp3/blob/master/handson1.md)

一般的なWeb3層アプリケーションで考えると可用性の担保としてセッションステートの分離、DBの可用構成がある。  
k8s上でもセッションステートをアプリから分離してRedisなどで可用性を担保することになる。こうすることでユーザーのセッションをキープしたままサービスの更新などが可能になる。

Apache KafkaはLAMPスタックから置き換わったSMACKスタックを担うOSSである。  
2011年にLinkedInから公開されたオープンソースの分散メッセージングシステム。Kafkaによってメッセージの中継を複数のPodで分散して行う。それぞれのPodはお互いにバックアップを取るので、可用性、回復性が実現できる。

ハンズオンではスペック不足？で動かない人が多かったらしいが、自分のMacbook2015は手順通りにやれば特に問題なかった。

# アプリケーションプラットフォーム層の可用性

##リリースと可用性
アプリケーションプラットフォーム層の可用性にはリリースも関連してくる。  
変更は障害のリスクであり、バグを含んでいないものをミスなくリリースできるCI/CD環境が用意されている必要がある。  
Istioなどを利用すればセンシティブなカナリーデプロイメントも可能

# Kubernetes層・ハードウェア層の可用性

## 巨人の肩に乗る

[Building High-Availability Clusters](https://kubernetes.io/docs/admin/high-availability/building/)

一言でいうと自前で可用性の高いk8sのクラスター構成を用意するのはめちゃくちゃめんどくさい。
ハードウェアにしても自前でマルチゾーン構成を用意するはよほどの資金力が必要。クラウドサービスに頼るのがよさそう。



# ハンズオン2: chaoskubeによるランダムなPodのシャットダウン

- [CNDJP #3 ハンズオンチュートリアル Part 2](https://github.com/oracle-japan/cndjp3/blob/master/handson2.md)
- [【AWS re:Invent 2017】「カオスエンジニアリング」について](https://www.skyarch.net/blog/?p=13294)

ハンズオン1で作成したKafkaを利用したチャットアプリに対して、k8s Podをランダムにダウンさせるカオスエンジニアリングを実施した。
k8sのPodに対してカオスエンジニアリングを行うにはchaoskubeがあるようだ。

https://github.com/linki/chaoskube

Macの場合は`brew install httpd`で`ab`コマンドが利用できるようになる。
たしかにPodは落ちているのだが、メッセージの欠損なくチャットが継続された。


# 終わってみて
実は最初の一時間くらいは`vagrant`上のk8sクラスターが全然起動しなくて（正確にいうとssh接続が出来なくて`vagrant up`が成功しなくて）、
話をちゃんと聞けていなかった。ベースがなくて初めて聞く事ばかりだし、ちょっと動かなくなるとお手上げになってしまうので、
自己学習が必要だなと感じる。Kodecataとかで勉強しないと。

[Learn Kubernetes using Interactive Browser-Based Scenarios](https://www.katacoda.com/courses/kubernetes)


次回はIstio。名前しか知らないのでちょっとでもわかるようになるといいなー。

# 参考

- [Susan Fowler氏に聞く - 実用段階のマイクロサービスとは](https://www.infoq.com/jp/news/2017/03/production-ready-microservices)
- [The Four Layers Of Microservice Architecture](https://www.susanjfowler.com/blog/2016/12/18/the-four-layers-of-microservice-architecture)
- [マイクロサービスのトレードオフ](http://postd.cc/microservice-trade-offs/)
- [SPOF(単一障害点）](https://ja.wikipedia.org/wiki/%E5%8D%98%E4%B8%80%E9%9A%9C%E5%AE%B3%E7%82%B9)
- [Pattern: Health Check API](http://microservices.io/patterns/observability/health-check-api.html)
- [Pattern: Circuit Breaker](http://microservices.io/patterns/reliability/circuit-breaker.html)
- [Microsoft Docs:回復性のパターン](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/category/resiliency)
  - [正常性エンドポイントの監視パターン](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/health-endpoint-monitoring)
  - [サーキット ブレーカー パターン](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/circuit-breaker)
- [YouTube:Building Fault Tolerant Microservices](https://www.youtube.com/watch?v=pKO33eMwXRs)
- [Istio: マイクロサービスのためのサービスメッシュ](https://www.infoq.com/jp/news/2017/06/istio)
- [Apache Kafka](http://kafka.apache.org/)
- [Apache Kafkaに入門した](https://deeeet.com/writing/2015/09/01/apache-kafka/)
- [Java Clientで入門する Apache Kafka](https://www.slideshare.net/techblogyahoo/java-client-apache-kafka-jjugccc-ccce2)
- [linki/chaoskube](https://github.com/linki/chaoskube)
- [【AWS re:Invent 2017】「カオスエンジニアリング」について](https://www.skyarch.net/blog/?p=13294)
- [Learn Kubernetes using Interactive Browser-Based Scenarios](https://www.katacoda.com/courses/kubernetes)

# 関連
- [Cloud Native Developers JP 第1回勉強会 参加メモ #cndjp1](/2017/11/23/cndjp1/)
- [[k8s]Cloud Native Developers JP 第2回勉強会 参加メモ #cndjp2](/2017/12/18/kubernetes-in-production-cndjp2/)


