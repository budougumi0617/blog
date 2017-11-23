+++
title= "Cloud Native Developers JP 第1回勉強会 参加メモ #cndjp1"
date= 2017-11-23T16:31:12+09:00
draft = false
slug = ""
categories = ["report", "kubernetes"]
tags = ["kubernetes"]
author = "budougumi0617"
+++

Cloud Native Developers JP(cndjp)のハンズオンに参加し、`Kubernetes`について勉強してきた。
クラウド素人の自分でもローカルにクラスター構成を作成、アプリのスケールイン、スケールアウトまで試せたので良かった。

|||
|---|---|
|URL|https://cnd.connpass.com/event/69971/|
|会場|日本オラクル青山センター|
|日時|2017/11/22(水) 19:00 〜 22:00|
|資料|https://www.slideshare.net/charlier-shoe/kubernetes-serverless-cndjp1/|
|ハッシュタグ| #cndjp1 |


# TL;DR
だいぶ基礎的なところからフォローしてくださっていた。「クラウド勉強してみたいけど、どこから手をつければ/どう手をつければいいんだろう？」と言った状態の自分にとってちょうどいいレベル感だったので、次回も参加したい。

- `Kubernetes`(以下k8s)の概要と基本的用語の理解
	- https://www.slideshare.net/charlier-shoe/kubernetes-serverless-cndjp1/
- OSSのサーバーレスフレームワーク、`Fn Project`の紹介
	- http://fnproject.io/
- `Vagrant` + `Virtual Box`上に1マスターノード、2メンバーノードのk8sクラスター環境を構築。
- サンプルアプリのPodをデプロイ、APIサーバー経由でアクセス
	- https://github.com/pires/kubernetes-vagrant-coreos-cluster
	- https://github.com/oracle-japan/cndjp1/blob/master/handson/handson1.md
- サンプルアプリに対してNodePortモードでServiceを構成
- 手動でPodのスケールアウト/スケールインを実施
- `stern`で複数Podのログを取得
	- https://github.com/wercker/stern
	- https://github.com/oracle-japan/cndjp1/blob/master/handson/handson2.md


Cloud Native Developers JPはClound NativeなOSSスタック（以下の図）の勉強会とのこと。  
https://github.com/cncf/landscape

![Cloud Native Landscape](https://raw.githubusercontent.com/cncf/landscape/master/landscape/CloudNativeLandscape_latest.jpg)


以下メモ

---


# Kubernetesとは
https://kubernetes.io/

Kubernetes(k8s)はgoogleが公開しているDockerを使った自動デプロイ、スケーリングなどのコンテナの運用自動化のためのオープンソースのプラットフォーム。最近は[Azureがフルマネージドを言及](http://jp.techcrunch.com/2017/10/25/20171024microsoft-new-azure-kontainer-service-puts-its-focus-on-kubernetes/)したり、[Docker本体が統合を発表](http://www.publickey1.jp/blog/17/dockerkubernetesdockercon_eu_2017.html)したり、オーケストレーションツールのデファクトスタンダードになりつつある。

# K8sの基本的な用語

今回のハンズオンで意識できるようになったのは以下。DaemonSetとかも用語説明のときにでてきたが、手を動かすとき関係なかった（はず）

## kubectl
k8sを操作するためのコマンドラインツール。コマンドをk8sのREST API呼び出しに変換する。k8sの操作は基本REST APIベースになっているのかな？いかにもWebツールだ。

## Node
クラスターに属するマシンを表すオブジェクト。クラスターの管理を担当するマスターノードとアプリケーションを稼働させるメンバーノードがある。今回で言うとVagrant上のVMになる。


## Pod
Node内で稼働するコンテナのセット。今回で言うとサンプルアプリ。ライフサイクルを操作するときの単位になるため、複数コンテナでひとつのPodを構成するときもある。

## Deployment
Podのライフサイクルを制御するオブジェクト。Podを起動したり、スケールするときに指定する。

## Service
Podへのアクセスの制御を行うオブジェクト。各DeploymentのPodへのルーティング、ロード・バランシングの役割を持つ。スケール操作したPodへのアクセスはServiceがよしなにしてくれるらしい。

## Label/Label Selector
k8sオブジェクトを管理。jsonで書ける。メタデータ、タグとして使う。Labelベースで操作すればPodが増減したときもいい感じにロードバランスなどが働く。

その他クラスター要素について深く知りたいならば、以下などを参考にすれば良いとのこと。

Creating a Custom Cluster from Scratch  
https://kubernetes.io/docs/getting-started-guides/scratch/

# Fn Project

http://fnproject.io/  
https://github.com/fnproject/fn

サーバレスアプリを作るためのフレームワーク。今年のJavaOneで発表された。Open & Easy。
マルチ言語サポートでDockerがあれば動く。Fn Server上に各Functionが乗るDocker in Dokcer構成。
トリガーがキックされるたびに対応するファンクションのコンテナを立ち上げる。
コンテナ管理はCLIで隠蔽される。
Fn Flowを使うと、複数のファンクションの連結を手続き的記述で実現（現状Javaのみ
コンテナで全部動いているので、k8sと相性が良いはずらしい。


# stern

https://github.com/wercker/stern

複数のPodのログを簡単に確認できるツール。色分けとかもしてくれるし確かに見やすかった！

[kubernetes使いは全員 stern を導入すべき](https://medium.com/@lestrrat/kubernetes%E4%BD%BF%E3%81%84%E3%81%AF%E5%85%A8%E5%93%A1-stern-%E3%82%92%E5%B0%8E%E5%85%A5%E3%81%99%E3%81%B9%E3%81%8D-bc9d3eb2c321)


--- 

# その他

- 練習問題は `kubectl describe pods | egrep "Node:|^Name:"`って感じで確認できた。
- kubectlはzsh(とbash)使いならば `source <(kubectl completion zsh)`を実行しておけば補完もかなり効く。
- Podを削除するときは`kubectl delete pod $POD_NAME`。
- 終わるときは`kubectl delete pods --all`して`vagrant halt`でよいかな？

