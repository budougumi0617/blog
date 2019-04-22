+++
title= "[Go] CicleCI2.1でgo modのデータを共有しながら複数ジョブを実行する"
date= 2019-04-21T15:31:49+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go", "circleci"]
tags = ["ci", "golang", "circleci"]
keywords = ["CircleCI", "Go", "golang", "継続的インテグレーション", "CI"]
twitterImage = "logos/circleci.png"
+++

GitHub上のGoのリポジトリに対して継続的インテグレーション(CI)を行なう場合、CircleCIやTravisCIを使うのが一般的だろう。  
CicrcleCI2.1でGo Modulesを使いながらマルチJobを定義したWorkflowを定義する。  
`attach_workspace`を使ってジョブ間のデータ共有をするのにひと手間必要だった。

<!--more-->

# TL;DR
- CircleCI2.0はWorkflowを使って複数Jobを平行実行することができる
- `save_cache`を使うとWorkflowを実行するたびにModuleをダウンロードせずに済む
- `persist_to_workspace`を使うと各ジョブで毎回Moduleをダウンロードせずに済む
  - `attach_workspace`するときに`user:root`の設定が必要な場合がある
- めんどくさいときは[`@__timakin__`](https://twitter.com/__timakin__)さんのCircleCI Orbを使えば簡単

- timakin/go-module | CircleCI Orb Registry
  - https://circleci.com/orbs/registry/orb/timakin/go-module

文中で部分参照している`.circleci/config.yml`の全文やCircleCIの実行結果は以下になる。

- [github.com/budougumi0617/.circleci/config.yml](https://github.com/budougumi0617/caww/blob/bc7896079c715e0dfe172033a28e3392bfa18468/.circleci/config.yml)
- https://circleci.com/gh/budougumi0617/caww

# CircleCIでGoのCIを定義する
まずCircleCIでGoを実行するときの基本的な設定方法は公式ページで参照することができる。

- Language Guide: Go | CircleCI
  - https://circleci.com/docs/2.0/language-go/

（`GO111MODULE=on`としていた場合、）Go1.11からGo Moudlesが利用できるようになっているので、
テストなどを実行する前に依存パッケージをダウンロードする必要がある。

また、CIはなるべく短時間で実行を終わらせたいので、サンプルの設定とは別に以下の設定をして実行時間を短くしていく。

## Workflowを利用して処理を平行化する。
CircleCI2.0から、1回のCIの実行の中にWorkflowと言う概念と、Jobという概念が加わった。

- Using Workflows to Schedule Jobs
  - https://circleci.com/docs/2.0/workflows/

一回のCI実行がWorkflowという単位になり、Workflowの中に複数のJobというステージを作成することができる。  
そして、「ジョブBはジョブAが完了したあとに実行する」、「ジョブBとジョブCは同時に実行できる」などの設定ができるようになる。  
今回はWorkflowの中に以下のジョブを定義する。

- `setup`ジョブ
  - `go mod dwonload`を行なって外部ライブラリのコードをインストールしておくジョブ
- `e2e`ジョブ
  - `go test`を実行するジョブ
- `lint`ジョブ
  - `go lint`などを実行して静的解析を行なうジョブ

`setup`ジョブで`go mod download`などの他のジョブが実行するために必要な処理を全て済ませておく（他のジョブにダウンロードしたデータの共有などをするには、後述する`persist_to_workspace`の設定が必要）。
`e2e`ジョブと`lint`ジョブは互いの実行結果を必要としないので、同時に実行することができる。
並行に実行できるようにしておくことで、Workflow全体の実行時間を短縮することができる。

3つのジョブを定義してジョブ間の依存関係を定義すると`.circleci/config.yml`は以下のような設定になる。

```yaml
version: 2.1
executors:
  default:
    docker:
      # CircleCI Go images available at: https://hub.docker.com/r/circleci/golang/
      - image: circleci/golang:1.12.2
    working_directory: /go/src/github.com/budougumi0617/caww
    environment:
      - GO111MODULE: "on"

jobs:
  # 事前にもろもろのダウンロードなどを済ませておくジョブ
  setup:
    executor:
      name: default
    steps:
      - checkout
      - run:
          name: Vendoring
          command: go mod download

  # e2eテストを実行するジョブ
  e2e:
    executor:
      name: default
    steps:
      # テストの実行、結果のアップロードなど...

  # linterを実行するジョブ。e2eジョブとは並行で実行することができる
  lint:
    executor:
      name: default
    steps:
      # 静的解析の実行など...

workflows:
  build-and-test:
    jobs:
      - setup
      - e2e:
          # e2e jobはsetupジョブが完了後に実行される
          requires:
            - setup
      - lint:
          # lint jobはsetupジョブが完了後に実行される
          requires:
            - setup
```

## save_cacheを使ってgo mod downloadの結果を保存しておく
毎回Worlflowを実行するたびにmodulesのダウンロードをするのは時間がかかる。ここで、CircleCIには前回のWorkflowで使ったデータをキャッシュしておく機能がある。

- Caching Dependencies | CircleCI
  - https://circleci.com/docs/2.0/caching/

`go.mod`ファイルの内容が同じ（checksumした結果が同じ）だったら前回のWorkflowでダウンロードした結果を再利用してダウンロード時間の短縮を行なう。

```yaml
jobs:
  setup:
    executor:
      name: default
    steps:
      - checkout
      # もし前回のWorkflow時に保存したキャッシュが利用できるなら再利用する
      - restore_cache:
          name: Restore go modules cache
          keys:
              - v1-mod-{{ .Branch }}-{{ checksum "go.mod" }}
      - run:
          name: Vendoring
          command: go mod download
      # ダウンロードしたmoduleをCircleCIにキャッシュとして保存しておく
      - save_cache:
          name: Save go modules cache
          key: v1-mod-{{ .Branch }}-{{ checksum "go.mod" }}
          paths:
              - /go/pkg/mod/cache
```

`go mod`を使ってダウンロードされるmoduleは`$GOPATH/pkg/mod/cache`に保存される。CircleCI公式のGo実行イメージの`$GOPATH`は`/go`なので`/go/pkg/mod/cache`をキャッシュすることになる。

## persist_to_workspaceを使ってジョブ間でデータを共有する
`e2e`ジョブの前に`setup`ジョブを実行して`go mod download`をしておくだけでは、`e2e`ジョブで`setup`ジョブがダウンロードした情報を利用することができない。
ジョブ間でデータを共有するためには`persist_to_workspace`と`attach_workspace`を利用する。

- persist_to_workspace | Configuring CircleCI
  - https://circleci.com/docs/2.0/configuration-reference/#persist_to_workspace

```yaml
jobs:
  setup:
    executor:
      name: default
    steps:
      # 前述したgo mod downloadなどの処理
      - persist_to_workspace:
          root: /go
          paths:
            - src
            - bin
            - pkg/mod/cache

  e2e:
    executor:
      name: with_mysql
    steps:
      - attach_workspace:
          at: /go
      # テストの実行、結果のアップロードなど...
```

`setup`ジョブに`persist_to_workspace`を設定し、ジョブ間で共有したいデータの場所を設定しておく。`e2e`ジョブ(`lint`ジョブ)に`attach_workspace`を設定し、共有したいデータをマウントする場所を設定する。
前述したとおり`$GOPATH`は`/go`なので、`/go`配下のデータを全て共有する。

---

**2019/04/22追記**  
`go mod`のキャッシュだけを再利用するだけならば`restore_cache`だけすればよいという指摘を頂いた。たしかに…！
<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="ja" dir="ltr">これ setup で save_cache してるので他のジョブで restore_cache すれば go mod のキャッシュ使えるようになるはずなんだけどなー / 他2件のコメント <a href="https://t.co/XE6f6XeSy8">https://t.co/XE6f6XeSy8</a> “[Go] CicleCI2.1でgo modのデータを共有しながら複数ジョブを実行する - My External Storage” <a href="https://t.co/hLS0LBcQ3I">https://t.co/hLS0LBcQ3I</a></p>&mdash; とにかく明るいへっくす (@codehex) <a href="https://twitter.com/codehex/status/1120114987928711169?ref_src=twsrc%5Etfw">April 21, 2019</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

追記ここまで。

---


# attach_workspaceしようとするとError applying workspace layer for job ...: Error extracting tarball ...: exit status 2
これでうまくいくかと思いきや、実際にCircleCIを実行してみると`attach_workspace`がうまくいかなかった。

- https://circleci.com/gh/budougumi0617/caww/13

```bash
Downloading workspace layers
  workflows/workspaces/2a7d4f28-c4f5-477a-90ab-190809d6e4f9/0/c5acd0fa-dc09-4e94-83b8-5e1f2e8a0e96/0/109.tar.gz - 171 MB
Applying workspace layers
  c5acd0fa-dc09-4e94-83b8-5e1f2e8a0e96

Error applying workspace layer for job c5acd0fa-dc09-4e94-83b8-5e1f2e8a0e96: Error extracting tarball /tmp/workspace-layer-c5acd0fa-dc09-4e94-83b8-5e1f2e8a0e96985056964: exit status 2
```

ググっても、データ量が多すぎるとこのようなエラーになる、というような解説が出てくるが、171MBくらいでなることはないらしい。

# ジョブをrootユーザで実行するようにすれば良い
結論から言うと、単純に`circleci`ユーザで解凍したデータを`root`ユーザ所有の`/go`ディレクトリ以下に展開しようとしていたのが原因だった。
`circleci/golang`イメージでは、`/go`は`root`ユーザ所有らしい。

`docker`の設定に`user: root`としておくと、各Jobの実行ユーザが`root`ユーザになるので`attach_workspace`が成功するようになった。

```yaml
version: 2.1
executors:
  default:
    docker:
      # CircleCI Go images available at: https://hub.docker.com/r/circleci/golang/
      - image: circleci/golang:1.12.2
        user: root
    working_directory: /go/src/github.com/budougumi0617/caww
    environment:
      - GO111MODULE: "on"
```

# 完成した.circleci/config.yml
以上の設定を全て行った`.circleci/config.yml`は以下のようになる。

```yaml
version: 2.1
executors:
  default:
    docker:
      - image: circleci/golang:1.12.2
        user: root
    working_directory: /go/src/github.com/budougumi0617/caww
    environment:
      - GO111MODULE: "on"

jobs:
  setup:
    executor:
      name: default
    steps:
      - checkout
      - restore_cache:
          name: Restore go modules cache
          keys:
              - v1-mod-{{ .Branch }}-{{ checksum "go.mod" }}
      - run:
          name: Vendoring
          command: go mod download
      - save_cache:
          name: Save go modules cache
          key: v1-mod-{{ .Branch }}-{{ checksum "go.mod" }}
          paths:
              - /go/pkg/mod/cache
      - persist_to_workspace:
          root: /go
          paths:
            - src
            - bin
            - pkg/mod/cache

  e2e:
    executor:
      name: default
    steps:
      - attach_workspace:
          at: /go
      # 以下テスト実行など

  # lint
  lint:
    executor:
      name: default
    steps:
      - attach_workspace:
          at: /go
      # 以下静的解析実行など

workflows:
  build-and-test:
    jobs:
      - setup
      - e2e:
          requires:
            - setup
```

実際に動いているの結果を見たい場合は以下を確認すること（GitHub上の設定には`MySQL`を使う設定なども含まれている）。

- [github.com/budougumi0617/.circleci/config.yml](https://github.com/budougumi0617/caww/blob/bc7896079c715e0dfe172033a28e3392bfa18468/.circleci/config.yml)
- https://circleci.com/gh/budougumi0617/caww

# 終わりに
publicなGitHubリポジトリにコードをプッシュしておけば、CircleCIなど多くのサービスを利用してOSS開発ができる。  
積極的に利用していきたい。なお、「上記の設定をいろいろなリポジトリで毎回するのがめんどくさい」というような方には[`@__timakin__`](https://twitter.com/__timakin__)さんのCircleCI Orbを使えば簡単に設定できる。

- [timakin/go-module | CircleCI Orb Registry](https://circleci.com/orbs/registry/orb/timakin/go-module)

# 参考
- [github.com/budougumi0617/.circleci/config.yml](https://github.com/budougumi0617/caww/blob/bc7896079c715e0dfe172033a28e3392bfa18468/.circleci/config.yml)
- https://circleci.com/gh/budougumi0617/caww
- [Language Guide: Go | CircleCI](https://circleci.com/docs/2.0/language-go/)
- [Caching Dependencies | CircleCI](https://circleci.com/docs/2.0/caching/)
- [persist_to_workspace | Configuring CircleCI](https://circleci.com/docs/2.0/configuration-reference/#persist_to_workspace)
- [timakin/go-module | CircleCI Orb Registry](https://circleci.com/orbs/registry/orb/timakin/go-module)


