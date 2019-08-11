+++
title= "[Helm] 配列の要素数だけリソースを繰り返し定義する"
date= 2019-08-11T19:17:22+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["helm", "kubernetes"]
tags = ["k8s", "helm", "kubenetes"]
keywords = ["k8s", "Helm", "Kubernetes"]
twitterImage = "logos/helm.png"
+++


Kubernetes(k8s)のパッケージマネージャとしてHelmが存在する。  
今回はHelmで独自のChartを作るときに配列の要素数だけリソースを繰り返し定義するようにしてみたのでその方法をまとめておく。

- https://helm.sh/

<!--more-->

# TL;DR
- prefix文字列を含んだ配列情報を使って名前が少し異なるリソース定義を複数作りたかった
- HelmはGoのtext/templateをベースにしたYAMLの定義をすることができる
  - [The Chart Template Developer’s Guide](https://helm.sh/docs/chart_template_guide/)
  - [Package template](https://golang.org/pkg/text/template/)
- `range`構文と`if`構文を使うことで定義できた。


```yaml
{{- range $i, $server := .Values.servers -}}
{{- if ne $i 0 }}
---
{{- end }}
apiVersion: v1
kind: Service
metadata:
  name: = {{ $server }}-{{ include "repeated-service.fullname" $ }}
  labels:
# ...
```

なお、文中で利用しているサンプルコードは以下。

- https://github.com/budougumi0617/til/tree/master/helm/repeated-service

## 環境情報
本記事で利用しているHelm関連のバージョンは以下の通り。
helm-tillerを使っているため、k8sクラスタ上にTiller serverは存在しない。

```bash
$ tiller -version
v2.14.3
$ helm version
Client: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
Error: Get http://localhost:8080/api/v1/namespaces/kube-system/pods?labelSelector=app%3Dhelm%2Cname%3Dtiller: dial tcp [::1]:8080: connect: connection refused
$ minikube version
minikube version: v1.2.0
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.7", GitCommit:"0c38c362511b20a098d7cd855f1314dad92c2780", GitTreeState:"clean", BuildDate:"2018-08-20T10:09:03Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:32:14Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}

```

# 背景
最近k8sの定義も自分が行なうことになり、Helmなどのキャッチアップを始めた。  
ここで、あるコンテナイメージをk8s上で動かしたいのだが、テストの数だけ`Service`/`Deployment`を別々にしてコンテナイメージを動かしたくなった。  
このテストの数は今後増えることもあるのでDRYなYAML定義をしたかった。そのため、テスト名の配列を用意して、配列の長さと同じ数だけリソースを繰り返し定義するようなHelm Chartを書きたくなった。


# 事前準備
## helm-tillerの準備
必須ではないが、tillerサーバを作らないで済むようにhelm-tillerをインストールしておく。

```bash
$ helm plugin install https://github.com/rimusz/helm-tiller
```
## minikubeを使ってローカルでHelmがdry-runできるようにしておく
Helmは`--dry-run`を使うとHelm Chartがどんなマニフェストに展開されるか確認することができる。
しかし、`--dry-run`時にもクラスター自体は存在する必要がある。`minikube`を使うと比較的簡単にk8sクラスターをローカルPC上に作成できる（Dockerの for MacなどでKubernetesを構築しても良いかもしれない。）。

- Install Minikube | Kubernetes
  - https://kubernetes.io/docs/tasks/tools/install-minikube/

## ベースとなるHelmを準備する
まずはじめにベースとしてChartを作成しておく。本記事では`helm create`で作成されるYAMLを例に進める。

```bash
$ helm create repeated-service
$ cd repeated-service
```

`helm create`で作成される`templates/service.yaml`の内容は以下の通り。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "repeated-service.fullname" . }}
  labels:
{{ include "repeated-service.labels" . | indent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: {{ include "repeated-service.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
```


このYAMLを使うと、以下のようなリソース定義となる。

```bash
$ helm tiller run -- helm install --debug --dry-run --name sample .
Installed Helm version v2.14.3
Installed Tiller version v2.14.3

...

# Source: repeated-service/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: sample-repeated-service
  labels:
    app.kubernetes.io/name: repeated-service
    helm.sh/chart: repeated-service-0.1.0
    app.kubernetes.io/instance: sample
    app.kubernetes.io/version: "1.0"
    app.kubernetes.io/managed-by: Tiller
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: repeated-service
    app.kubernetes.io/instance: sample

...

```

いま、`values.yaml`に以下のような配列を追加したとする。

```yaml
# values.yaml
servers:
  - alice
  - bob
  - charlie
  - dave
```

そして先ほどの`repeated-service/templates/service.yaml`を修正して、Valuesに定義した配列によって以下のように繰り返しマニフェストが出力される`service.yaml`を定義してみる。

```yaml
# templates/service.yaml 以下のようにserversリストの要素だけServiceを出力するようにしたい。
# servers[0]のalice-sample-repeated-serviceの定義
apiVersion: v1
kind: Service
metadata:
  name: alice-sample-repeated-service
# ...
---
# servers[1]のbob-sample-repeated-serviceの定義
apiVersion: v1
kind: Service
metadata:
  name: bob-sample-repeated-service
# ...
---
# ...
```

# 配列の要素を使って繰り返しリソースを定義する
実際に`Service`を配列分だけ宣言するように修正してみる。

## HelmではGoのtext/template構文を使える
修正は変数とループ処理を使って行なう。Helmでループ処理などをYAMLで行なうには以下の記事を参考にする。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://helm.sh/" data-iframely-url="//cdn.iframe.ly/PzLUCYM"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

また、HelmはGo製のツールでその記法はGoの`text/template`パッケージに準拠している。

- Package template
  - https://golang.org/pkg/text/template/

なのでこの2つの情報を使ってYAML中にループを含んだりしながら定義できる。

実際に編集した`templates/service.yaml`は以下のようになる。

```diff
+{{- range $i, $server := .Values.servers -}}
+{{- if ne $i 0 }}
+---
+{{- end }}
 apiVersion: v1
 kind: Service
 metadata:
-  name: {{ include "repeated-service.fullname" . }}
+  name: = {{ $server }}-{{ include "repeated-service.fullname" $ }}
   labels:
-{{ include "repeated-service.labels" . | indent 4 }}
+{{ include "repeated-service.labels" $ | indent 4 }}
 spec:
-  type: {{ .Values.service.type }}
+  type: {{ $.Values.service.type }}
   ports:
-    - port: {{ .Values.service.port }}
+    - port: {{ $.Values.service.port }}
       targetPort: http
       protocol: TCP
       name: http
   selector:
-    app.kubernetes.io/name: {{ include "repeated-service.name" . }}
-    app.kubernetes.io/instance: {{ .Release.Name }}
+    app.kubernetes.io/name: {{ $server }}-{{ include "repeated-service.name" $ }}
+    app.kubernetes.io/instance: {{ $server }}-{{ $.Release.Name }}
+{{- end -}}
```

`range`構文を使うと、ループ処理が行える。`if A == B`のような比較文は`eq`を使って書ける。  
また、ループ処理中はスコープが変わり、`.`の参照先がループ変数になるので、`$`を使ってルートコンテキストから`Values`などを参照している。  
（なお、`labels`の`{{ include "repeated-service.labels" $ | indent 4 }}`部分の名前付きテンプレートの利用を止めないとラベルの定義が運用的に問題だが、今回はブログ記事なので省略している）

## 名前付きテンプレートにしないのか？
名前付きテンプレートを使う場合、`include`文の引数としてで渡せるオブジェクトが1つしかない。  
そのため、いちいち`dict`文を使って`$server`変数、`Service`、`Values`などを1つのオブジェクトに詰め直す必要があり、冗長な書き方になるので止めている。


```yaml
# こんな書き方をしないといけなくなってしまう
{{- include "repeated-service.service" (dict "Bank" $bank "Realease" .Realese "Values" .Values) -}}
 
```

## helm installの実行結果

実際にこの定義を使って`helm install --dry-run`してみた結果が以下。


```bash
$ helm tiller run -- helm install --debug --dry-run --name sample .
Installed Helm version v2.14.3
Installed Tiller version v2.14.3
Helm and Tiller are the same version!
Starting Tiller...
Tiller namespace: kube-system
Running: helm install --dry-run --name sample .

...

# Source: repeated-service/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: = alice-sample-repeated-service
  labels:
    app.kubernetes.io/name: repeated-service
    helm.sh/chart: repeated-service-0.1.0
    app.kubernetes.io/instance: sample
    app.kubernetes.io/version: "1.0"
    app.kubernetes.io/managed-by: Tiller
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: alice-repeated-service
    app.kubernetes.io/instance: alice-sample
---
# Source: repeated-service/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: = bob-sample-repeated-service
  labels:
    app.kubernetes.io/name: repeated-service
    helm.sh/chart: repeated-service-0.1.0
    app.kubernetes.io/instance: sample
    app.kubernetes.io/version: "1.0"
    app.kubernetes.io/managed-by: Tiller
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: bob-repeated-service
    app.kubernetes.io/instance: bob-sample

...

```

無事`.Values.servers`の値を使って複数の`Service`定義が作成されている。`Deployment`も同様に書き換えることでやりたいことは実現できる。

# 終わりに
最初は名前付きテンプレートで書こうと思って試行錯誤していたのだが、本記事のように書くほうがシンプルだったのでこのようにした
（実際公式Chartでも名前付きテンプレートへの代入で複雑な変数定義をすることはほとんどないらしい）。  
まだ出来上がるマニフェストの理解などは曖昧なところがあるので引き続きk8s関連のキャッチアップを続ける。

# 参考
- [The Chart Template Developer’s Guide](https://helm.sh/docs/chart_template_guide/)
- [Package template](https://golang.org/pkg/text/template/)

