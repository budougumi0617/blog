+++
title= "[dkl] 実行中のDocker ContainerやKubernetes Podを一覧して選択、Execするツールを作った"
date= 2019-09-28T18:05:18+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["docker", "kubernetes", "go"]
tags = ["golang", "docker", "kubenetes", "k8s"]
keywords = ["golang", "Docker", "Kubernetes", "k8s", "OSS"]
twitterImage = "twittercard.png"
+++

同僚がPythonで作ったOSSをGoに移植してみた。

```bash
$ dkl -h
Usage of dkl:
  -d -docker
        list docker containers and exec selected container.
  -k -kubernetes
        list pods and exec selected pod.
  -v  -version
        print version information and quit.
```

<!--more-->

# TL;DR
- Docker ContainerやKubenetes Podにログインするには、一度起動中のContainer（Pod）を一覧する必要がある
    - さらに名前をコピーして`docker(kubectl) exec`コマンドを実行する必要がある
- `dtt`コマンドは1回のコマンドで一覧を表示、`exec`コマンドで選択したContainer（Pod）にログインする。
    - Pythonで作られている。
- Goで書き直したのが`dkl`コマンド
- `promptui`を使ったので一覧からの選択部分はリッチ

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/dkl" data-iframely-url="//cdn.iframe.ly/qGCYud2"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

![demo](/2019/09/28_dkl_demo.gif)

# dklの概要
Dockerコンテナにログインしたいとき、大抵は`docker ps`コマンドを実行し、コンテナの名前を確認し、`docker exec`コマンドを実行することになる。
これは、Kubernetes上のPodにログインするときもほぼ同様の操作になる。

`dkl`コマンドは上記をラクするために同僚がPythonで作っていたツールをGoで書き直したものだ。

https://github.com/ymizushi/dtt
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/ymizushi/dtt" data-iframely-url="//cdn.iframe.ly/tKMaJHE"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

https://github.com/budougumi0617/dkl
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/dkl" data-iframely-url="//cdn.iframe.ly/qGCYud2"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


`dkl`コマンドを使うと、Dockerコンテナ（あるいはKubernetes Pod）一覧を表示し、選択したコンテナにログインすることができる。

インストールには`Brew`を利用できる。

```bash
$ brew install budougum0617/tap/dkl
```

2019/09/28時点の使い方は次の通り。

```bash
$ dkl -h
Usage of dkl:
  -d    list docker containers and exec selected container.
  -docker
        list docker containers and exec selected container.
  -k    list pods and exec selected pod.
  -kubernetes
        list pods and exec selected pod.
  -v    print version information and quit.
  -version
        print version information and quit.
```

作り込んていないので、一覧できる情報は少ないが、`dkl`コマンドを引数なしで動かすと、実行中のコンテナ一覧を表示する。
ひとまず名前とImage情報を表示している。

```bash
$ dkl
Use the arrow keys to navigate: ↓ ↑ → ←  and / toggles search
Docker running...?
  🐋 /mysql (mysql:5.7, sha256:98455b9624a96e32b353297bb312913b6bbd62ac195cea2c7dd477209ba572d6)
     /mysql8 (mysql:8.0, sha256:91dadee7afeebe274c51104d572ab6a2dc0ae97473f71afc57fbfd48c0ceb8aa)

--------- Image ----------
Name:           ["/mysql"]
Image:          mysql:5.7
ImageID:        sha256:98455b9624a96e32b353297bb312913b6bbd62ac195cea2c7dd477209ba572d6
```

この中から選択したコンテナに`docker exec`コマンドを実行して、インタラクティブシェルを開始する。

KubernetesのPodの場合は以下のような情報を表示する。

```bash
dkl -k
Use the arrow keys to navigate: ↓ ↑ → ←  and / toggles search
Kubernetes pod running...?
  ⎈ nginx (default Running)
    coredns-5c98db65d4-fxttt (kube-system Running)
    coredns-5c98db65d4-lv9gh (kube-system Running)
    etcd-minikube (kube-system Running)
    kube-addon-manager-minikube (kube-system Running)
    kube-apiserver-minikube (kube-system Running)
    kube-controller-manager-minikube (kube-system Running)
    kube-proxy-kchl6 (kube-system Running)
    kube-scheduler-minikube (kube-system Running)
    storage-provisioner (kube-system Running)

--------- Pod ----------
Name:               "nginx"
Namespacege:        default
Status:             Running
Age:                114h51m50.205832s
```
こちらも同様に選択したPodに`kubectl exec`コマンドを実行する。

対象を選択する部分は`manifoldco/promptui`パッケージをそのまま利用しているので、絞り込み検索もできる。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/manifoldco/promptui" data-iframely-url="//cdn.iframe.ly/nlTsYKh"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

技術書典の準備の隙間でやっていたので、雑なところもあるが、ひとまずちゃんと動くものができてよかった。

# 特徴
機能的には前述の機能しかない。Goで実装してよかった特徴としては以下があげられる。

- シングルバイナリ・マルチプラットフォーム提供が簡単
- Go Releaserを使ったので、デプロイが簡単
    - https://hacktoberfest.digitalocean.com/
- `manifoldco/promptui`を使ったのでインタラクティブな絞り込みが簡単にできた
- DockerもKubernetesもGo SDKが公開されているので、APIの呼び出しが簡単

不出来な点としては、実装を省略するため、ログインは`docker`（`kubectl`）コマンドを外部プロセスとして実行している。
本当はSDKの`exec`用のAPIを利用したかったが、時間がたりなかった。

# 終わりに
ひととりカタチにはなったのだが、あまり作り込めていない。
具体的には、思いつくところだけで以下のような改善点がある。

- 構造化が雑
- testableではない
- `Pod`の`Age`で表示している時間が分かりにくい（標準パッケージだと`time.Duration`をいい感じに表示できない）
- 実行時に`exec`で呼び出す実行コマンドの指定に`ls -la`などが指定できない
- ログインでは外部コマンドを実行しているので、`docker`、`kubectl`コマンドに依存している

他にも作りたいOSSがあったり、来月は`Hacktoberfest`もあるのですぐにはできないが、引き続きカイゼンしていきたい。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://hacktoberfest.digitalocean.com/" data-iframely-url="//cdn.iframe.ly/jZlOhKY?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 参考

- https://github.com/ymizushi/dtt
- https://github.com/budougumi0617/dkl
- https://goreleaser.com/
- https://hacktoberfest.digitalocean.com/
