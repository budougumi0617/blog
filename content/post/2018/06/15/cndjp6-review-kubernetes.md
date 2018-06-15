+++
title= "[k8s]隅からスミまでKubernetes総復習 - cndjpシーズン1まとめ参加メモ #cndjp #k8s"
date= 2018-06-15T22:12:45+09:00
draft = false
slug = ""
categories = ["report","kubernetes"]
tags = ["k8s","cndjp"]
author = "budougumi0617"
+++

Cloud Native Developers JP(cndjp) の第6回目に参加してきたので、自分メモ。

|||
|---|---|
|URL|https://cnd.connpass.com/event/89854/|
|会場|オラクル青山センター 13F セミナールーム|
|日時|2018/06/14(木) 18:30 〜 21:30|
|資料1|[Kubernetes応用](https://speakerdeck.com/hhiroshell/kubernetesying-yong)|
|資料2|[Kubernetesマニアック](https://speakerdeck.com/hhiroshell/kubernetesmaniatuku)|
|ハッシュタグ| [#cndjp](https://twitter.com/hashtag/cndjp)|


# TL;DR
知っていると便利あるいはハマらないKubernetes(k8s)の細かい設定を学ぶことができた。

- Podの起動時と終了時に行われる・行える処理
  - InitContainerによる初期化処理を指定できる
  - Container Lifecycle Hooksによる終了時と開始時にフック処理
  - Liveness/Readiness Probesによる稼働監視
- Podに対する制限の付与
  - Resource Limit/Request リソースに対するコンテナの要件や制限を指定
  - Resource Quota namespace単位でリソース使用量の制限
  - Pod Security Policy セキュリティ関係の制約
  - Network Policy 他のエンドポイントに通信を行うときのポリシー
- Podの配置先の制御
  - nodeSelector Nodeに貼ったLablelで配置先を制御
  - Node Affinity nodeSelectorより条件の表現力が高い
  - Pod Affinity 他のPodの稼働状況で新たなPodの配置先を制御
  - Taint/Toleration Nodeに条件を設定して特定の条件を満たしたPodだけスケジューリング
- Kubernetesの拡張
  - 独自Objectを定義・利用することができる
  - プラグインを利用することでNetwork、DeviceやStorageの拡張ができる

# 所感
終了後「入門k8s」を一通り眺め直したが、どれも書いていないことばかりだったので「こういう事ができる」と知れることができて良かった。  
Pod起動時に前処理を挟むのは環境変数などの設定などで絶対使いたくなる気がする。  
また、ポリシーの付与は組織として多種なサービスをKubernetes上で構築するときに必要になるのかなと思った。  
プラグインなどの仕組みがあるおかげでk8sはOSSとして複数のベンダー、デバイス上で動かすことができるのだろう。

以下、概要と関連リンクのメモ

# Podの起動時と終了時に行われる・行える処理
## Pod Phase
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase

- Podの状態を示す`PodStatus`には`phase`フィールドがある
- `phase`には起動前の`Pending`フェーズと起動したことを示す`Running`フェーズが存在する
- `Pending`フェーズでPodの作成やNodeへの配置が行われる
- `Running`フェーズでコンテナのENTRY POINTなどの起動が始まる



## `Pending`フェーズでできること
`Pending`フェーズでInitContainersという前処理用のコンテナが起動している。

### Init Containers
https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

- InitContainerはユーザーが用意しなくても動いているものもある
- InitContainerは複数使うこともできる。その場合はシーケンシャルに実行される
- IPアドレスの収集や外部サービスとの接続を待機するなどの処理を行う

## `Running`フェーズでできること
Container Lifecycle Hookを使うとコンテナの生成、破棄をフックして特定の処理が実行できる。

### Container Lifecycle Hooks
https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/

- `PostStart hook`でコンテナ生成時の処理を定義できる
- `PreStop hook`でコンテナ終了前の処理を定義できる
  - それぞれスクリプトを動かす`Exec`と特定のリクエストを発行する`HTTP`の2種類を指定できる

### Container Probes
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes  
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/

コンテナの起動が成功したのか、コンテナが正しく動いているかの監視定義を決められる。

- `Readiness probe` コンテナがトラフィックを受け入れられる状態になったかの判定する
  - コンテナの起動 = サービスレディではないため
- `Liveness probe` コンテナが"生きている"か確認するための条件
  - コンテナインスタンスがあってもサービスが稼働しているかはわからない
- それぞれ`Manifest`に条件を指定することができる
  - `ExecAction` 指定したコマンドが終了コード0で終わるか
  - `HTTPGetAction` 指定のエンドポイントが200ステータスを返すか
  - `TCPSocketAction` TCPソケットのコネクションを張れるか


### 廃棄のときに行える処理

**Kubernetes: 詳解 Pods の終了**  
https://qiita.com/superbrothers/items/3ac78daba3560ea406b2

- Podの終了イベントが起きたとき
  - `Service`の`Endpoint`リストから除外され、Trafficが送られなくなる
  - `Prestop hook`が起動される。
- `PreStop hook`が終わらないときはSIGTERM or SIGKILLで強制終了になってしまうこともある。
  - どちらのシグナルで終了するかは`gracePeriodSecconds`で決まる
  - 決め打ちでシグナルを実装しないこと
  - `Manifest`で指定したり`kubectl`から指定可能



# ポリシーによる制御
Podに適用できるポリシーは主に4つ

## Resource limits/ Resource requests
https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container

- 消費リソースの上限/下限を設定できる
  - CPU
- 配置前
  - 内包するコンテナ群のRsource requestの合計をまかなえる余剰がないNodeにはPodがスケジューリングされない
- 配置後
  - limitを超えないように設定することができる

## Local Ephemeral Storage
https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#local-ephemeral-storage

- Nodeのrootパーティションの利用量に対してLimit/requestを設定できる

## Extended Resource
https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#extended-resources

- k8sが未サポートの独自管理したいリソースを追加できる。
  - GPUとか。
  - Nodeごとに指定することもできる

## Resource Quota
https://kubernetes.io/docs/concepts/policy/resource-quotas/

- `namespace`単位で設定可能なリソース使用量の制限

## Pod Security Policy
https://kubernetes.io/docs/concepts/policy/pod-security-policy/

- コンテナ内にいろいろポリシーを制限することができる
- チームでクラスターを共有する運用でユーザーごとにできることを制限したいとき
- `plivileged`なコンテナを作成する操作はクラスター管理者のみに許可(監視系のコンテナを各Nodeにするなど）

## Network Policies
https://kubernetes.io/docs/concepts/services-networking/network-policies/

- Podが他のエンドポイントに通信を行うときのポリシーを設定
- Network Policyリソースを作成することで有効化する
- Network Policyはそれをサポートするnetwork providerを利用する必要がある
- 例えば`flannel`単体では使えず、Calicoと組みあわせる必要がある。
- `Istio`を使っているとそのまま使うことができる。

# Podの配置の制御
https://kubernetes.io/docs/concepts/configuration/assign-pod-node/

## nodeSelector
https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector

- Nodeにあらかじめ指定されたラベルにマッチするNodeからPodが配置される
- 暗黙的にNodeに設定されているLabelもある。マネージドサービスを使っていると特に。

## Node Affinity/anti-Affinity
https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity

- 妥協的に配置したり、PodのLabelを条件に使える
- あるLabelがついているPodと同じNodeには載せないとか。
- required/preferredの条件がある。
- preferredにはweightを指定することができて、weightの合計値が大きいほどそのNodeに配置されやすい。

## Pod Affinity
- すでに稼働しているPodに基づいて新しいPodを配置する条件にできる
 - 例えば、Redisクラスターを分散配置したり、クライアントアプリをredisコンテナと同じ場所に乗せたり。

## TaintとToleration
https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/

- TaintsはNodeに設定される値。TolerationsはPodに指定する値
- それぞれの条件がマッチするPodだけNodeに乗る。
- 例えば、特定のユーザーだけにNodeを使わせたいケース
- 例えば、特殊なHWを必要なPodだけに使わせるケース

# Kubernetesの拡張

- ほとんどのケースでは、`Deployment`などの`Controller`オブジェクトを作成することにより、結果的にPodが作成される。
- 各オブジェクトに対応する`Controller`がある。
- 目的にユースケースに合わせて、Podをよしなに制御してくれるのが標準Controller
- `Deployment`, `ReplicaSet`などの標準Controllerを動かしているデーモンプロセスがcontroller-manager
 - Custom Controllerを作れば独自オブエジェクトが作れる

## Custom ResourceとKubernetes APIの拡張
https://kubernetes.io/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/  
https://www.ianlewis.org/jp/extending-kubernetes-ja

- kubernetesに独自のリソースを追加して利用することができる。
  - Kubernetes APIの拡張
  - Custom Objectの作成
  - Custom Controllerの拡張
- Kubernetes APIの拡張には二通りの方法がある
  - CustomResourceDefiinition(CRD)
    - 独自のリソースタイプをAPI Serverに登録することCustom Objectを取り扱えるようになる
  - Aggregation Layerの追加
    - 標準API serverと独立した独自のAPI機能をクラスターに追加できる
- Custom Objectの作成
  - `Manifest`を定義する
- Custom Controllerの拡張
  - コードジェネレータを使えばCustom Controllerで利用するクライアントライブラリの一部を自動生成できる
    - https://github.com/kubernetes/code-generator

**KubernetesのCRD(Custom Resource Definition)とカスタムコントローラーの作成**  
https://qiita.com/__Attsun__/items/785008ef970ad82c679c

## インフラの拡張
https://kubernetes.io/docs/concepts/extend-kubernetes/extend-cluster/

- 特殊なデバイスを利用するための拡張ができる
- Device PluginやNetwork Pluginとか
- デバイスプラグイン本体の準備（デバイスベンダーからの情報に従う）
- k8sのDevice Pluginを有効化
- Podからデバイスの利用を宣言
- Network Plugin(CNI)
  - CNCFのプロジェクトとして仕様策定が進んでいる
  - https://www.cncf.io/blog/2017/05/23/cncf-hosts-container-networkinpg-interface-cni/
- Network Plugin(kubenet)
  - https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
  - GKEなどでクラスターを生成すると自動で構成される

# Container Runtime Interface(CRI)
https://github.com/kubernetes/kubernetes/blob/242a97307b34076d5d8f5bbeb154fa4d97c9ef1d/docs/devel/container-runtime-interface.md

- kubeletからPod及びContainerを操作するための共通インターフェース
- cri-oはOCI標準に準拠したランタイムを実行する仕組み

# その他の拡張
- スケジューリングの制御周り
  - Configure Multiple Schedulers
    - https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
- オートスケーリングやストレージのプラグインもできる。
  - Horizontal Pod Autoscaler
    - https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
  - Introducing Container Storage Interface (CSI) Alpha for Kubernetes
    - https://kubernetes.io/blog/2018/01/introducing-container-storage-interface/

# 関連
- [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)
- [Cloud Native Developers JP 第1回勉強会 参加メモ #cndjp1](/2017/11/23/cndjp1/)
- [[k8s]Cloud Native Developers JP 第2回勉強会 参加メモ #cndjp2](/2017/12/18/kubernetes-in-production-cndjp2/)
- [[k8s]Cloud Native Developers JP 第3回勉強会 #cndjp3 参加メモ](/2018/02/03/kubernetes-with-availability-cndjp3/)
- [Kubernetes Network Deep Dive!（Istioもあるよ） cndjp第四回参加メモ #cndjp4](/2018/04/01/kubernetes-network-deep-dive-cndjp4/)
- [[k8s]あつまれ！ CI/CDツール大集合！ - cndjp第5回 参加メモ #cndjp5](/2018/05/02/cndjp5-cicd-tools/)

