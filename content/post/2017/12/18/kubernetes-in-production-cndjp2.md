+++
title= "[k8s]Cloud Native Developers JP 第2回勉強会 参加メモ #cndjp2"
date= 2017-12-18T09:00:00+09:00
draft = false
slug = ""
categories = ["report", "kubernetes"]
tags = ["kubernetes","k8s","cndjp"]
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
|ハッシュタグ| [#cndjp2](https://twitter.com/search?q=%23cndjp2) |

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

[Container Engine(Docker)のlogging driver](https://docs.docker.com/engine/admin/logging/overview/)がファイルとして出力している。[logrotate](https://linux.die.net/man/8/logrotate)でローテートされている。そのログをagentがbackendに転送する。GKEの場合はbackendにstackdriverが動いていて、よしなに出来る。

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





# PersistentVolume/PersistentVolumeClaimを利用した永続化機能の使い方
https://kubernetes.io/docs/concepts/storage/persistent-volumes/  

k8sでは`PersistentVolume`/`PersistentVolumeClaim`を利用して、コンテナ内で利用できるボリュームを多様に用意できる。ハンズオン1では仮想ボリュームを作成し、3つのコンテナで共有させていた。

## Volume
https://kubernetes.io/docs/concepts/storage/volumes/

Podにマウント可能なボリュームを定義する情報。
次に説明する`PersistentVolume`/`PersistentVolumeClaim`を利用すれば、動的に永続ストレージを用いる事もできる。
downloadAPI。コンテナ内のアプリケーションが自分のラベル情報とかを参照できる。configMapは一般的な設定情報。secretsは機密性の高い情報。などなど。

## PersistentVolume
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes

k8s上の永続ストレージのオブジェクト。ストレージの種類、権限や容量などの情報を定義する。

## PersistentVolumeClaim
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


## ローリングアップデート
Podをひとつずつアップデートすることで、無停止アップデートを行う手法。Deploymentオブジェクトをりようすると自動的にアップデート管理をしてくれる。

**Performing a Rolling Update**  
https://kubernetes.io/docs/tutorials/kubernetes-basics/update-intro/

**Run a Stateless Application Using a Deployment**  
https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/

Replicationコントローラを使う方法もあるらしいが、公式でもDeploymentの利用を促している。

**Perform Rolling Update Using a Replication Controller**  
https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/

## カナリーアップデート（カナリアリリース）

カナリーアップデートは物理サーバに依存しないいかにもクラウドなアップデート方法。現行バージョン稼働中の裏で新バージョンのコンテナを起動し、順次新バージョンに切り替えていく。２つのバージョンを同時稼働しながらアップデートを行うので、ロールバックが簡単。k8sでやる場合はLabelとLabelSelectorをうまく使う。


**Canary deployments**  
https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments

**Canary Release with Kubernetes**  
https://hackernoon.com/canary-release-with-kubernetes-1b732f2832ac

# k8s運用時に行うべきセキュリティの設定方法
コンテナレジストリ、SSHアクセス、各コンテナに対する権限付与など、いくつかセキュリティについて考えるところがある。基本的には外部とのやりとりをするところ。

- apiserver API(Kubernetesのリソースを管理するAPIサーバー)の認証/許可機能を利用する
	- https://kubernetes.io/docs/admin/authorization/
- kubelet API(	Podを起動し管理するエージェント)の認証/許可機能を利用する
	- https://kubernetes.io/docs/admin/kubelet-authentication-authorization/
- ノードへのSSHアクセスを制限する。
- コンテナレジストリのクレデンシャルをSecretオブジェクトで保持し、Podに生で置かない
- コンテナに制限をかけておく
	- 利用可能リソース
	- ネットワークアクセス
	- 実行権限
	- ノード制限
- k8sに対する操作の監査ログを有効にしておく
- etc...


**authorization**  
https://kubernetes.io/docs/admin/authorization/

**Kubelet authentication/authorization**  
https://kubernetes.io/docs/admin/kubelet-authentication-authorization/

**Kubernetes: 構成コンポーネント一覧**  
https://qiita.com/tkusumi/items/c2a92cd52bfdb9edd613


# ハンズオン2: PersistentVolumeを利用した`Node.js`/`MySQOL`アプリケーションを`Ingress`でクラスタ外部に公開する

https://github.com/oracle-japan/cndjp2/blob/master/handson2.md


1. mySQLのデータ領域用に、hostPath(ノードのローカルボリューム)のPersistentVolumeを作成する
	2. [deployment/db-pv-hostpath.yaml](https://github.com/oracle-japan/cndjp2/blob/master/koa-sample/deployment/db-pv-hostpath.yaml)
2. データベースのユーザー情報保存場所として、Secretを作成する
	3. [deployment/db-secret.yaml](https://github.com/oracle-japan/cndjp2/blob/master/koa-sample/deployment/db-secret.yaml)
3. 上記を利用するDBコンテナを起動する
	4. [deployment/db-deployment.yaml](https://github.com/oracle-japan/cndjp2/blob/master/koa-sample/deployment/db-deployment.yaml)
4. Node.jsコンテナのServiceオブジェクト、Deploymentオブジェクトを作成する
	5. [deployment/ap-deployment.yaml](https://github.com/oracle-japan/cndjp2/blob/master/koa-sample/deployment/ap-deployment.yaml)
5. ルーティングが見つからなかった時用のデフォルトのバックエンドを構成する
	6. [deployment/web-default-backend.yaml](https://github.com/oracle-japan/cndjp2/blob/master/koa-sample/deployment/web-default-backend.yaml)
7. NodeにLabelを設定(`kubectl label nodes 172.17.8.102 role=front`)して、デプロイ先を固定してから`NGINX Ingress Controller`をデプロイする
	8. [deployment/web-rc-default.yaml](https://github.com/oracle-japan/cndjp2/blob/master/koa-sample/deployment/web-rc-default.yaml)
9. Ingressオブジェクトを構成する。
	10. [deployment/web-ingress.yaml](https://github.com/oracle-japan/cndjp2/blob/master/koa-sample/deployment/web-ingress.yaml)


ハンズオンのマニフェストファイルの中身はちゃんと読めていないので、まだ別に調べてまとめたい。

