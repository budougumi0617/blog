+++
title= "[Go]次世代イメージcimg/goとcircleci/go Orbsを使った2020年版CircleCIの環境構築"
date= 2020-06-08T09:05:58+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go","circleci"]
tags = ["circleci","golang","CI","test"]
keywords = ["CircleCI","CI","Go言語","自動化","cimg/go","circleci/go"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

2020年になって、CircleCIのCI用の公式ベースイメージは`cimg/base`派生になった。  
ただ、2020/06/07現在、Go向けの言語別公式ガイドの中身は古いままである。  
`cimg/go`を使ったGo向けのCircleCIの設定をまとめる。

<!--more-->

# TL;DR
- CircleCIで利用するコンテナイメージに次世代版が登場した。
    - https://hub.docker.com/r/cimg/go
- 直接使わなくても、Orbsが便利
    - https://circleci.com/orbs/registry/orb/circleci/go
    - `go/mod-download-cached`でキャッシュとGo Modules周りがすぐ解決。
- `go-junit-report`を`go get`しなくてもよい
    - `gotestsum`コマンドがデフォルトインストールされている
- `GOPATH`は変更されているので注意する
    - `/home/circleci/go`

最小構成のサンプルコードは次のとおり。

```yaml
version: 2.1
orbs:
  go: circleci/go@1.2.0
jobs:
  build:
    executor:
      name: go/default
      tag: '1.14.4'
    environment:
      TEST_RESULTS: /tmp/test-results
    steps:
      - checkout
      - run: mkdir -p $TEST_RESULTS
      - go/mod-download-cached
      - run: gotestsum --junitfile ${TEST_RESULTS}/unit-tests.xml -- -p 6 -race -cover ./...
      - run: go build ./...
      - store_artifacts:
          path: /tmp/test-results
          destination: raw-test-output
      - store_test_results:
          path: /tmp/test-results
```


サンプルリポジトリは以下。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/cimg_go" data-iframely-url="//cdn.iframe.ly/6jm5N1X"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

なお、動作確認で使ったツールの各バージョン情報は次のとおり。

||version|
|---|---|
|Go|1.14.4|
|cimg/go image|1.14.4|
|circleci/go orbs|1.2.0|

# 次世代型イメージのCirclCIの公式イメージ cimg/go 

2020年2月にCircleCIは`cimg/base`を基にした次世代版のCI用のベースイメージを提供しはじめることをアナウンスした。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://circleci.com/blog/announcing-our-next-generation-convenience-images-smaller-faster-more-deterministic/" data-iframely-url="//cdn.iframe.ly/DKcOtBl?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


そして5月にはGo用の`cimg/go`が正式リリースされた。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://circleci.com/changelog/" data-iframely-url="//cdn.iframe.ly/9pTZ1zb?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

リリース文のとおり、これは従来の`circleci/golang`に取って代わるものだ。

> This is a direct replacement for the legacy CircleCI Go image (circleci/golang). You can find this image on Docker Hub, and the source code & documentation on GitHub.

`cimg/xxxx`系は`circleci/xxx`系と比較して、起動時間が高速化されていてリリーススケジュールも安定しているらしい。  
（ただ今回作成したサンプルリポジトリではそこまで高速化の恩恵を感じることはできなかった）。  
Go以外にもRubyやRust用のイメージもリリースされている。

ただ、2020/06/07現在CircleCI公式ページの言語別セットアップガイドは古い`circleci/golang`イメージを使った`.circleci/config.yml`の設定のままだ。

- https://circleci.com/docs/2.0/language-go/

なので、2020/06/07時点の最新のCircleCIの設定方法をまとめる。

# cimg/goをつかったCircleCIの設定
`cimg/go`イメージと、`circleci/go` orbsを使った環境設定方法をまとめる。
## circleci/golangイメージを使った従来のCircleCIの設定
まず、公式のGo用の環境構築に記載されている従来の設定は次のとおり（CircleCIの設定ファイルバージョンとGoのバージョンは変更している）。
明示的にGo Modulesを実行していない。
テスト結果を保存するために`jstemmer/go-junit-report`を`go get`していたり冗長な処理が多い。

```yaml
version: 2.1
jobs:
  build:
    docker:
      - image: circleci/golang:1.14.4
    environment:
      TEST_RESULTS: /tmp/test-results
    steps:
      - checkout
      - run: mkdir -p $TEST_RESULTS
      - restore_cache:
          keys:
            - v1-pkg-cache
      - run: go get github.com/jstemmer/go-junit-report
      - run:
          name: Run unit tests
          command: |
            trap "go-junit-report <${TEST_RESULTS}/go-test.out > ${TEST_RESULTS}/go-test-report.xml" EXIT
            go test -v -p 6 -race -cover ./... | tee ${TEST_RESULTS}/go-test.out
      - run: go build ./...
      - save_cache:
          key: v1-pkg-cache
          paths:
            - "/go/pkg"
      - store_artifacts:
          path: /tmp/test-results
          destination: raw-test-output
      - store_test_results:
          path: /tmp/test-results
```

## cimg/goとcircleci/go orbsを使った2020年版の設定
次世代イメージを使った環境設定は次のとおりになる。
`cimg/go`イメージをベースに作成された公式orbsイメージを使えばかなりシンプルに書けるようになる。

```yaml
version: 2.1
orbs:
  go: circleci/go@1.2.0 # https://circleci.com/orbs/registry/orb/circleci/go

jobs:
  build:
    # mysqlイメージなども使いたいならばdockerタグでcimg/goを指定すればよいだろう。
    executor:
      name: go/default # the base image is cimg/go
      tag: '1.14.4'

    environment:
      TEST_RESULTS: /tmp/test-results

    steps:
      - checkout
      - run: mkdir -p $TEST_RESULTS
      - go/mod-download-cached

      - run:
          name: Run unit tests
          command: |
            gotestsum --junitfile ${TEST_RESULTS}/unit-tests.xml -- -p 6 -race -cover ./...
      - run: go build ./...

      - store_artifacts:
          path: /tmp/test-results
          destination: raw-test-output

      - store_test_results:
          path: /tmp/test-results
```

CircleCIの実行結果は次の通り。
- https://app.circleci.com/pipelines/github/budougumi0617/cimg_go/12/workflows/ededb9df-62e9-4186-bf90-26d6a772d117/jobs/12/steps

YAML上の設定で変わった点は次のとおりだ。

- OrbsでGo Modulesまわりが簡潔に宣言できる
- コンテナに最初から入っている`gotestsum`でテスト結果のXMLを出力できる

## Go Modulesとcache周りの設定
`circleci/go` orbsを使えばGo Modules周りの設定は一行で完了する。  
このOrbsは`cimg/go`イメージをベースに作られている。

- https://circleci.com/orbs/registry/orb/circleci/go

`go/mod-download-cached`はOrbsに定義された次の処理を一括で行なうステップなので、これだけでCircleCIキャッシュのロード、`go mod download`、キャッシュの保存が完了する。

- `go/load-cache`
- `go/mod-download`
- `go/save-cache`

もちろん個別に利用することもできる。

## gotestsumを使ったテスト結果の集計

従来の設定では、`go-junit-report`を使ってテスト結果をXMLで集計していた。
```yaml
- run:
  name: Run unit tests
  command: |
    trap "go-junit-report <${TEST_RESULTS}/go-test.out > ${TEST_RESULTS}/go-test-report.xml" EXIT
    go test -v -p 6 -race -cover ./... | tee ${TEST_RESULTS}/go-test.out
```

`cimg/go`イメージには、`gotestsum`コマンドが標準で含められており、これを使えばわざわざ`go get`せずともXMLでテスト結果を集計できる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://circleci.com/blog/level-up-go-test-with-gotestsum/" data-iframely-url="//cdn.iframe.ly/yCFsBxM?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

そのため、次のような1行でテスト結果を集計しながらこちらで決めたオプションを使って`go test`が実行できる。

```yaml
- run: gotestsum --junitfile ${TEST_RESULTS}/unit-tests.xml -- -p 6 -race -cover ./...
```

ちゃんと設定できていると、ジョブ実行結果のテストタブでテスト結果が集計される。

- https://app.circleci.com/pipelines/github/budougumi0617/cimg_go/4/workflows/eda37ff8-0026-4805-b5b0-e70b2eb5712e/jobs/4

![テスト結果の集計](/2020/06/08_test_fail.png)

## cimg/goのGOPATH
既存のYAMLに手を加えず、`circleci/golang`から`cimg/go`にひとまず移行したいときもあるかもしれない。  
`circleci/golang`という指定を`cimg/go`に変更するだけでも動くのだが、`GOPATH`は変わっているので注意すること。  
`cimg/go`イメージの`GOPATH`は`/home/circleci/go`になっている。

# 終わりに
Orbsを使うことでだいぶYAMLがさっぱりした。
最近は流行りもありGitHub Actionsを使いがちだったが、CircleCIのキャッチアップができてよかった。


# 参考
- https://github.com/budougumi0617/cimg_go
  - https://app.circleci.com/pipelines/github/budougumi0617/cimg_go
- [Announcing our next-generation convenience images: smaller, faster, more deterministic | circleci Blog][announce]
- [CircleCI’s next-gen Go convenience image has been released][release_cimg_go]
- [cimg/go | Docker Hub][dockerhub]
- [circleci/go | CirceCI Orbs][orbs]
- [Level up go test with gotestsum | cricleci Blog][gotestsum]

[announce]: https://circleci.com/blog/announcing-our-next-generation-convenience-images-smaller-faster-more-deterministic/
[release_cimg_go]: https://circleci.com/changelog#circleci-s-next-gen-go-convenience-image-has-been-released
[dockerhub]: https://hub.docker.com/r/cimg/go 
[orbs]: https://circleci.com/orbs/registry/orb/circleci/go
[gotestsum]: https://circleci.com/blog/level-up-go-test-with-gotestsum/