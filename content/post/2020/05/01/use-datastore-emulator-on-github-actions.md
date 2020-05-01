+++
title= "GitHub Actions上でGCP Datastoreエミュレータを使ったテストを実行する"
date= 2020-05-01T08:42:01+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go","GitHub","GCP"]
tags = ["datastore","golang","gcp","github","actions"]
keywords = ["Go","Golang","Go言語","Datastore","GCP","GitHub","GitHub Actions"]
twitterImage = "logos/github.png"
+++

GCP Datastoreを扱うコードをGitHub Actions上でテストする方法をまとめた。

<!--more-->

# TL;DR
- GCP Datastoreはエミュレータが用意されており、コンテナを提供している人もいる
    - https://cloud.google.com/datastore/docs/tools/datastore-emulator
    - https://hub.docker.com/r/singularities/datastore-emulator/
- GitHub ActionsでDBや外部ストレージを模した環境を用意するときは`jobs.<job_id>.services`を使う
    - https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idservices
- GCPのGoクライアントは環境変数を設定しておくだけで自動的にエミュレータへ接続する
    - https://cloud.google.com/datastore/docs/tools/datastore-emulator#manually_setting_the_variables

```yaml
jobs:
  job-name:
    runs-on: ubuntu-latest
    services:
      datastore:
        image: singularities/datastore-emulator
        env:
          DATASTORE_LISTEN_ADDRESS: 0.0.0.0:18001
          DATASTORE_PROJECT_ID: budougumi0617
        ports:
          - 18001:18001
    env:
      DATASTORE_EMULATOR_HOST: localhost:18001
```

なお、引用しているGitHub Actions、GCPの説明は2020/05/01時点の説明である。


# GitHub Actionsの設定
さっそく、エミュレータについて確認し、Actinosの設定YAMLを書いていく。

## GCP Datastoreエミュレータ
GCP Datastoreはエミュレータが提供されている。

- https://cloud.google.com/datastore/docs/tools/datastore-emulator

JREと`gcloud`コマンドを用意するのは骨が折れるので、Docker Hubで公開されている次のGCP Datastoreエミュレータコンテナを利用する。

- https://hub.docker.com/r/singularities/datastore-emulator/

## ActionsでDockerコンテナを起動する

GitHub Actions上で任意のDockerコンテナを実行するには次の手順が用意されている。

- `jobs.<job_id>.container`
    - https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idcontainer
- `jobs.<job_id>.services`
    - https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idservices

`jobs.<job_id>.container`はそのジョブ自体を実行する環境を指定するオプションのため、`jobs.<job_id>.services`のほうを利用する。
`jobs.<job_id>.services`の説明自体にも、以下のような助言が記載されている。

> ワークフロー中のジョブのためのサービスコンテナをホストするために使われます。 サービスコンテナは、データベースやRedisのようなキャッシュサービスの作成に役立ちます。 ランナーは自動的にDockerネットワークを作成し、サービスコンテナのライフサイクルを管理します。

## サービスコンテナへの接続方法

サービスコンテナ自体への接続は次のページに記載されている。

- https://help.github.com/en/actions/configuring-and-managing-workflows/about-service-containers

今回のActionはランナーマシン上でジョブを実行するため、次の記載に記載されている接続方法になる。

> ジョブをランナーマシン上で直接実行する場合、サービスコンテナにはlocalhost:<port>もしくは127.0.0.1:<port>を使ってアクセスできます。 GitHubは、サービスコンテナからDockerホストへの通信を可能にするよう、コンテナネットワークを設定します。


## 設定したYAML
`jobs.<job_id>.services`を使い、PRに対してGoのテストを実行するYAMLの設定は次のとおりになる。


```yaml
# .github/workflows/test.yml
on: [pull_request]
name: test
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    services:
      datastore:
        image: singularities/datastore-emulator
        env:
          DATASTORE_LISTEN_ADDRESS: 0.0.0.0:18001
          DATASTORE_PROJECT_ID: budougumi0617
        ports:
          - 18001:18001
    env:
      DATASTORE_EMULATOR_HOST: localhost:18001
    steps:
    - name: Install Go
      if: success()
      uses: actions/setup-go@v1
      with:
        go-version: 1.13.x # GAE/Go 1.13環境を想定して
    - name: Checkout code
      uses: actions/checkout@v1
    - name: Use Cache
      uses: actions/cache@v1
      with:
        path: ~/go/pkg/mod
        key: ubuntu-latest-go-${{ hashFiles('**/go.sum') }}
        restore-keys: |
          ubuntu-latest-go-
    - name: Download Modules
      if: steps.cache.outputs.cache-hit != 'true'
      run: go mod download
    - name: Run tests
      run: go test -v -race -covermode=atomic
```

# テストコードの作成
GitHub Actions上でDatastoreエミュレータを起動する準備はできた。
実際にエミュレータを使ったテストを書く。

## エミュレータへの接続設定
`google-cloud-go`ライブラリからエミュレータへ接続するようにするには、環境変数を指定するだけで良い。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://cloud.google.com/datastore/docs/tools/datastore-emulator" data-iframely-url="//cdn.iframe.ly/fmWbAwE?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`google-cloud-go`ライブラリからの`datastore`クライアントの初期化処理を確認すると次のようになっている。

- https://github.com/googleapis/google-cloud-go/blob/c9d3eadce82c530f46cf3c09fc607e329affe4b2/datastore/datastore.go#L67-L78

```go
func NewClient(ctx context.Context, projectID string, opts ...option.ClientOption) (*Client, error) {
  var o []option.ClientOption
  if addr := os.Getenv("DATASTORE_EMULATOR_HOST"); addr != "" {
    o = []option.ClientOption{
      option.WithEndpoint(addr),
      option.WithoutAuthentication(),
      option.WithGRPCDialOption(grpc.WithInsecure()),
    }
  } else {
        // ...
```

よって、ActionsのYAMLファイル中で宣言した`DATASTORE_EMULATOR_HOST: localhost:18001`という環境変数設定があるだけでエミュレータへ接続してテストが実施される。

テストコードは簡単に以下のとおり。

```go
package gaebot

import (
  "context"
  "testing"

  "cloud.google.com/go/datastore"
  "go.mercari.io/datastore/clouddatastore"
)


type CloudDatastoreStruct struct {
	Test string
}

// Actionsの動作確認。
func TestCloudDatastore_Put(t *testing.T) {
  ctx := context.Background()
  cli, _ := datastore.NewClient(ctx, "budougumi0617")
  client, err := clouddatastore.FromClient(ctx, cli)
  if err != nil {
    t.Fatal(err.Error())
  }
  defer client.Close()

  key := client.IncompleteKey("CloudDatastoreStruct", nil)
  if _, err = client.Put(ctx, key, &CloudDatastoreStruct{"Hi!"}); err != nil {
    t.Fatal(err.Error())
  }
}
```

# 終わりに
`google-cloud-go`ライブラリのDatastore以外のクライアントも少し確認したところ、
Cloud Storage用のクライアントも環境変数を設定する方法でエミュレータへ接続するようになっていた。
外部接続先を変更するようなテストを書く機会は度々あるので、クライアントライブラリがこのような設計になっているのは開発者としても使いやすいなと思った。

また、Actionsの設定には`timeout-minutes: 10`という設定も追加している。これはジョブの実行時間を分単位で制限する設定だ。
タイムアウトを設定していないまま、間違った設定でエミュレータへ接続を試みるとジョブが終わらない。
Actionsの無料実行時間制限に達しないように、このような設定をしておくほうが安心だろう。

# 参考
- https://cloud.google.com/datastore/docs/tools/datastore-emulator
- https://hub.docker.com/r/singularities/datastore-emulator/
- https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idservices
