+++
title= "[k8s]Cloud Native Developers JP 第2回勉強会 参加メモ #cndjp1"
date= 2017-11-23T16:31:12+09:00
draft = false
slug = ""
categories = ["report", "kubernetes"]
tags = ["kubernetes","k8s"]
author = "budougumi0617"
+++

第2回Cloud Native Developers JP(cndjp)のハンズオンに参加し、`Kubernetes`(以下k8s)について勉強してきた。  
第1回は最初ということもあり、基本的な用語と、ローカルのk8sクラスター環境でポッドを起動し、スケールさせてみるレベルの内容だった。  
[Cloud Native Developers JP 第1回勉強会 参加メモ #cndjp1](https://budougumi0617.github.io/2017/11/23/cndjp1/)

が、今回はYAMLに定義された`PersistentVolume`オブジェクトを使ったアプリケーション構築など、内容が濃かった。  
自分はコンテナ周りの知識がほとんどない（動かせるがゼロから構築は出来ない）ので、どこから学んでいくべきか、後日改めて動かせる成果物と掘り下げるきっかけを得られた非常によい勉強会だった。  
ハンズオンの内容を動かすだけ動かせたが、それぞれの概念や挙動の理解は追いつかなかったので、調べながら当日の内容をまとめる。  



|||
|---|---|
|URL|https://cnd.connpass.com/event/73694/|
|会場|日本オラクル青山センター|
|日時|2017/12/18(月) 19:00 〜 22:00|
|資料1|https://www.slideshare.net/charlier-shoe/kubernetes-in-cndjp2|
|資料2|https://github.com/oracle-japan/cndjp2|
|ハッシュタグ| #cndjp2 |

# TL;DR

 - 前回の復習。ノード、ポッドなどのk8s用語の再確認。
 - ロギングのアーキテクチャ
 - ハンズオン1: マニフェストファイルを利用して作成する複数サイドカーを用いたロギングサンプル
	 - https://github.com/oracle-japan/cndjp2/blob/master/handson1.md
 - `PersistentVolume`/`PersistentVolumeClaim`を利用した永続化機能の使い方
 - コンテナのアップデート手法の紹介
 - k8s運用時に行うべきセキュリティの設定方法
 - ハンズオン2: PersistentVolumeを利用した`Node.js`/`MySQOL`アプリケーションを`Ingress`でクラスタ外部に公開する
	 - https://github.com/oracle-japan/cndjp2/blob/master/handson2.md



# 前回の復習

最初は`Node`や`Pod`、`Deployment`など、k8sの基本的な用語の確認だった。スライド資料の説明(P11-P19)や前回メモがあるので割愛。  
https://www.slideshare.net/charlier-shoe/kubernetes-in-cndjp2  
[Cloud Native Developers JP 第1回勉強会 参加メモ #cndjp1](https://budougumi0617.github.io/2017/11/23/cndjp1/)


# ロギングのアーキテクチャ

**Logging Architecture**  
https://kubernetes.io/docs/concepts/cluster-administration/logging/

[Container Engine(Docker)のlogging driver](https://docs.docker.com/engine/admin/logging/overview/)がファイルとして出力している。[logrotate](https://linux.die.net/man/8/logrotate)でローテートされている。そのログをagentがbackendに転送する。GKEの場合は`backend`にstackdriverが動いていて、よしなに出来る。

**Logging Using Stackdriver**  
https://kubernetes.io/docs/tasks/debug-application-cluster/logging-stackdriver/

GKE以外の環境では、agentに`fluentd`、backendに`Elasticsearch`が挙げられている。

**Cluster-level logging architectures**  
https://kubernetes.io/docs/concepts/cluster-administration/logging/#cluster-level-logging-architectures

# ハンズオン1: マニフェストファイルを利用して作成する複数サイドカーを用いたロギングサンプル

https://github.com/oracle-japan/cndjp2/blob/master/handson1.md  

以下のような構成のPodのマニフェストファイルを作成し、実際にローカルのクラスター上で動かしてみる。

```
Pod
┣━二種類のログを出力しつづけるcounterコンテナ
┣━ログを取るsidecarコンテナ1
┣━ログを取るsidecarコンテナ2
┗━3つのコンテナにマウントされる仮想ボリューム
```


## Manifest File
https://kubernetes.io/docs/concepts/configuration/overview/  
k8sでPodを作成する時の設定ファイルは、マニフェストファイルと呼ばれ、YAMLで作成する。(JSONでも可能なことは可能らしい)  


## sidecar
https://docs.microsoft.com/ja-jp/azure/architecture/patterns/sidecar  
sidecarはクラウドデザインパターンのひとつで、アプリケーションのコンポーネントを別のコンテナに分離して、カプセル化を行うこと。





# `PersistentVolume`/`PersistentVolumeClaim`を利用した永続化機能の使い方
https://kubernetes.io/docs/concepts/storage/persistent-volumes/  

k8sでは`PersistentVolume`/`PersistentVolumeClaim`を利用して、コンテナ内で利用できるボリュームを多様に用意できる。ハンズオン1では仮想ボリュームを作成し、3つのコンテナで共有させていた。

## `Volume`
https://kubernetes.io/docs/concepts/storage/volumes/

Podにマウント可能なボリュームを定義する情報。
次に説明する`PersistentVolume`/`PersistentVolumeClaim`を利用すれば、動的に永続ストレージを用いる事もできる。
downloadAPI。コンテナ内のアプリケーションが自分のラベル情報とかを参照できる。configMapは一般的な設定情報。secretsは機密性の高い情報。などなど。

## `PersistentVolume`
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes

k8s上の永続ストレージのオブジェクト。ストレージの種類、権限や容量などの情報を定義する。

## `PersistentVolumeClaim`
そのPodが求めるPersistentVolumeの要件を定義する。この要件を満たすPersistentVolumeが自動的に選択される。これでコンテナと永続化層が疎に定義できる。

Podのvolumesに`hostpath`などのVolumeではなく、PersistentVolumeClaimを指定する。  
別にPersistentVolumeClaimを定義し、Claimの要件を満たすPersitentVolumeを作成しておく。そうすると、自動的にそのVolumeがPodに選択される。

**kubernetes: pod(コンテナ)のディスク(volume)とパス(path)の指定方法**  
https://qiita.com/suzukihi724/items/9003b453ddfb1acd202a

**kubernetesでPersistent Volumesを使ってみる**  
https://ishiis.net/2017/01/08/kubernetes-storage/

**第6章 KUBERNETES におけるストレージのプロビジョニング**  
https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux_atomic_host/7/html/getting_started_with_containers/get_started_provisioning_storage_in_kubernetes

# コンテナのアップデート手法の紹介

k8s上で動いているコンテナを更新する方法がいくつかある。

# k8s運用時に行うべきセキュリティの設定方法


# ハンズオン2: PersistentVolumeを利用した`Node.js`/`MySQOL`アプリケーションを`Ingress`でクラスタ外部に公開する






アップデートの話。
ローリングアップデート。Doploymentオブジェクトを使ってコンテナを展開しておくと、ReplicaSetが自動的にアップデート管理をしてくれる。
カナリーデプロイメントはlabel, LabelSelectorをうまく使うと実現できる。カナリーとして新バージョンのコンテナを追加する際に、Servivceのルーティング対象に。。。指定する？
最終的に安定版のイメージを新バージョンに変更する。
セキュリティ上の考慮点。
クラスターレベル。外部とのやり取りの部分に認証認可機能を使う。
kublet APIの認証認可。匿名アクセスを無効化したりする（デフォルトは許可している。）認可もデフォルトでは全て許可している。apiserverにポリシーを継承するように設定できる。
apiserverの認証認可。よしなにいろいろな情報で。
NodeへのSSHのアクセスの制限をしたりもできる。コンテナレジストリへのアクセスはSecretｗ作成して、Podでそれを利用するように設定する。
Secretはkubctlのコマンドオプションとかで作れる。
コンテナレベルのセキュリティ。コンテナ自体の安全性を確保したい、コンテナからの影響を最小限にすることが必要。
イメージの生い立ちを確認しておくこと。不必要なパッケージが含まれていないことを確認すこと。とか。サポートがどうなっているか、とか。
コンテナの利用リソースを制限したりする。Resource quotaを設定して専有防止をする。Namespaceごとにquotaを指定しておく。
NetworkPolicyというオブジェクトを作って、適用するPodとか条件を設定する。
コンテナ実行時の権限も調整できる。Pod内のコンテナのユーザーを指定したり。あるコンテナにだけ設定したりもできる。
コンテナが稼働するノードを制限する。Nodeにもラベルをつけられる。特定のLabelが設定されたNodeのみにPodをデプロイ出来る
etcdにクラスタ全体の機密情報を保管させるが、そのバックアップは暗号化する。各種ログをちゃんととっておく。
次回はネットワーク周りを深掘りするぞ！サービスメッシュでk8s上にインテリジェントなネットワークを作るぞ！