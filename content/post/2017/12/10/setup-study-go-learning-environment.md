+++
title= "[Go]CircleCI2.0+Codecovで作るGo学習環境"
date= 2017-12-10T08:13:12+09:00
draft = false
slug = ""
categories = ["Go","CI"]
tags = ["CircleCI", "Go", "CodeCov","GoReportCard"]
author = "budougumi0617"
+++

[Goならわかるシステムプログラミング](http://amzn.to/2kNkTlZ)の写経、練習問題を解くにあたり、`Go`の軽いCI環境を作った。

# TL;DR

自己学習や写経だと、レビューしてもらう(第三者の評価を得る)機会がない。それでも、自分の書いたコードに最低限の品質担保をする。
基本的にPublicリポジトリならば全て無料。

- `GitHub`にプッシュするたびに`CircleCI`で自動ビルドを行う。
- ビルド時にリポジトリに同梱されているテストを自動で実行する。
- テストのカバレッジを計測し、`Codecov`で結果を可視化する。
- 静的解析の結果を`Go ReportCard`で可視化する。
- バッジで各サービスの結果がわかる。

以下の設定は次のリポジトリに対して行った。

https://github.com/budougumi0617/gsp

設定中に出てくる`budougumi0617/gsp`という部分は各自のリポジトリに合わせて修正していただきたい。

# `GitHub`にプッシュするたびに`CircleCI`で自動ビルドを行う
今回はCircleCI2.0を使ってみる。まずCircleCIでリポジトリに対するビルドを有効にする。
CircleCIにログイン後、左のメニューから「Projects」を選び「Add Project」ボタンをクリックする。

![circleci_add_project](/2017/12/circleci_add_project.png)

Go用の設定を選んで「Start Building」ボタンをクリックすると、とりあえずビルドが実行される。
これでプッシュのたびに自動ビルドが始まる。

![circleci_start_building](/2017/12/circleci_start_building.png)

# ビルド時にリポジトリに同梱されているテストプロジェクトが自動で実行される。
次に自分のリポジトリに`.circleci/config.yml`を作る。内容は`CircleCI`の`Go`用のサンプル設定ファイルとほぼ同じ。

ちゃんとしたCircleCi2.0のGoの設定をしたい場合は以下のQiita記事が参考になりそう。  
[CircleCI2.0を使ってGoで開発したツールをLint,UT,ビルドしてGithubにリリースする](https://qiita.com/tomiyan/items/6142113011243c5b5cd1)


```yml
# Golang CircleCI 2.0 configuration file
#
# Check https://circleci.com/docs/2.0/language-go/ for more details
version: 2
jobs:
  build:
    docker:
      - image: circleci/golang:1.9

    working_directory: /go/src/github.com/budougumi0617/gsp
    steps:
      - checkout

      # specify any bash command here prefixed with `run: `
      - run: go get github.com/pierrre/gotestcover
      - run: gotestcover -coverprofile=coverage.txt ./...
      - run: bash <(curl -s https://codecov.io/bash)
```


`working_directory`は自分のリポジトリ名にする。
`gotestcover`は複数パッケージのテストカバレッジをいい感じにまとめてくれる。写経などをしていると、リポジトリの中に複数パッケージが出来るので。
`gotestcover`でまとめたテストカバレッジの結果は`Codecov`のスクリプトを実行すれば自動的にCodecovに送信される。


# テストのカバレッジを計測し、`Codecov`で結果を可視化する。
上記設定をしたあと、以下のようなURLを確認すればカバレッジの結果が表示されるはずだ。

https://codecov.io/gh/budougumi0617/gsp

# 静的解析の結果を`Go ReportCard`で可視化する。
`Go ReportCard`はもろもろのgo toolの静的解析の結果や、複雑度、スペルミスまで解析してくれる。

https://goreportcard.com/

TOPページから自分のリポジトリのパスを入力して「Generate Report」しよう。


リポジトリを作ってから時間が経っているならば、既にページが存在しているかもしれない。

https://goreportcard.com/report/github.com/budougumi0617/gsp


# バッジで各サービスの結果がわかる。
あとはREADMEにもろもろのサービスのバッジのリンクをつけておけば完成だ。

CircleCIは以下のようなURLからリンクを発行できる。

https://circleci.com/gh/{USER_NAME}/{REPO_NAME}/edit#badges

Codecovは以下のようなリンクから。

https://codecov.io/gh/{USER_NAME}/{REPO_NAME}/settings/badge

Go ReportCardはページのバッジをクリックするとリンクが表示される。

![Get Badge](/2017/12/get_badge_link.png)

# 余談
以前は`CodeClimate`も使っていたのだが、今はバッジで`Go`のリポジトリの静的解析結果は表示してくれないようだ。
https://codeclimate.com/  
[Supported Languages for Maintainability](https://docs.codeclimate.com/docs/supported-languages-for-maintainability)  
Pluginにもろもろを書けば、Issueという形で`go lint`などの結果は管理してくれる。

[Available Analysis Plugins](https://docs.codeclimate.com/docs/list-of-engines)
