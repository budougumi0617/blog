+++
title= "2018年振り返り(GitHub編)"
date = 2018-12-15T19:54:15+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["poem"]
tags = ["Retrospective", "github"]
twitterImage = "twittercard.png"
+++

この記事は、[write-blog-every-week Advent Calendar 2018](https://adventar.org/calendars/2925)の15日目の記事。  
「毎週ブログを書く」では無いが「毎日GitHubにコミットする」ということを続けている。今年も１年の振り返りを行なうにあたり、GitHubの活動を振り返ってみた。

<!--more-->


# コントリビュート
[作ったPR](https://github.com/pulls?utf8=%E2%9C%93&q=is%3Apr+author%3Abudougumi0617+created%3A%3E%3D2018-01-01+is%3Apublic)はこんな感じだった。
相変わらずタイポ警察しているだけなのは良くない。ただ、ちょっと機能追加やバグフィックスのPRも作ったので去年よりは少し成長できた。

- aws/aws-lambda-go
  - https://github.com/aws/aws-lambda-go/pull/14
- grpc-ecosystem/grpc-gateway
  - https://github.com/grpc-ecosystem/grpc-gateway/pull/532
- go-chi/docgen
  - https://github.com/go-chi/docgen/pull/7
- google/go-cloud
  - https://github.com/google/go-cloud/pull/504

ここまでだいたいREADMEなどのタイポを直しているだけ。

- tenntenn/devquiz
  - https://github.com/tenntenn/devquiz/pull/1
  - golang.tokyoで使っているツールの修正
- golangtokyo/codelab
  - https://github.com/golangtokyo/codelab/pull/2
  - golang.tokyoのハンズオン資料の修正
- kojirock/write-blog-every-week-remind-go
  - https://github.com/kojirock/write-blog-every-week-remind-go/pull/4
  - https://github.com/kojirock/write-blog-every-week-remind-go/pull/6
  - [いつもお世話になっているslackチーム](https://kojirooooocks.hatenablog.com/entry/2018/09/21/223841)で使われているLambdaのコードの修正
- flatfisher/fish-go
  - https://github.com/flatfisher/fish-go/pull/1
  - qiitaで見かけたGoのコードで気になったところがあったので直した。

# OSS
OSSとして作ったリポジトリは以下。今年は2つくらいしかできてない。

## lsas
- https://github.com/budougumi0617/lsas
  - List view of aws autoscaling group.

[私のチームは月イチでOSS Fridayというイベントを行なっている](https://developers.freee.co.jp/entry/introducing-godate)。その中で作ったGoのCLIツール。
Auto Scaling GroupをWebのAWS Console画面のようにCLIから閲覧・検索できる。これはだいぶ社内で使っていてもらって、[同僚からPRももらえた](https://github.com/budougumi0617/lsas/pull/1)。
PRもらうと「ちゃんとしたOSS」って感じで嬉しい。勢いで作ったのと、[@RealGophersShip](https://twitter.com/RealGophersShip)にリリースインフォ流してもらえないかいろいろいじったせいで内部構造がぐちゃぐちゃになっている。もう少しリファクタリングしたら紹介記事を書く予定。
このリポジトリはbrew用の設定もしているので、そちらの勉強もできた。


## godecov
- https://github.com/budougumi0617/godecov
  - Unofficial Go wrapper library for the Codecov API.

CodeCovのAPIラッパーがないのでこれは作るしかないでしょ！と思って作り始めたリポジトリ。
JSONに対応するstruct定義してちょっとGETメソッド作るだけでしょーと思って始めたもの、結構辛いことになっていてあまり捗っていない。

- [[Go] JSON内の数字や文字列が混じった配列を構造体にUnmarshalする](/2018/11/23/unmarshal-json-array-in-go/)

# Dotfile
https://github.com/budougumi0617/dotfiles

.vimrcにReact+Redux関係の設定を増やした。今年はVimを触っている時間がかなり多かったので、だいぶこなれてきている気がする。
（と言いつつこの間やっとeasymotionを入れた）

# その他今年作ったリポジトリ
[created:>=2018-01-01 is:publicで絞った結果](https://github.com/budougumi0617?utf8=%E2%9C%93&tab=repositories&q=created%3A%3E%3D2018-01-01+is%3Apublic&type=&language=)

## sandbox-aws-lambda-go
https://github.com/budougumi0617/sandbox-aws-lambda-go

AWS LambdaにGoが来た！ということで触ってみたリポジトリ。

- [AWS Lambda Go早めぐり(LambdaContext, Logging, Error...)](/2018/01/17/hello-aws-lambda-go/)

## sandbox-grpc
https://github.com/budougumi0617/sandbox-grpc

去年の冬からgRPCを触り始めたのでそのときの勉強用リポジトリ。

- [[Go]gRPC GatewayでgRPCに対するREST APIを自動生成する](/2018/02/03/grpc-gateway-for-rest-api/)
- [[Go]gomockを使ったgRPCのテスト](/2018/01/21/try-gomock-on-grpc-go/)

## simple-json-api-by-chi
https://github.com/budougumi0617/simple-json-api-by-chi

業務でGoのWebサーバーを書くことになったのでまずchiを触ってみていた。

- [[Go]go-chiで作ったREST APIの定義からMarkdownやJSONを動的に出力する](/2018/03/11/go-chi-generates-routing-map/)

## udemy_react
https://github.com/budougumi0617/udemy_react

React勉強してみよう！ということでUdemyの写経をしていた。

## react-golang
https://github.com/budougumi0617/react-golang

フロントエンドをReactで、バックエンドをGoで書いてみる挑戦をしていたリポジトリ。いちおうそれぞれを個別のDockerで動かして疎通するところまではやった。
初夏のリポジトリなので、フロントエンドサーバの設定はイチからやりなおしたほうが良さそう…

## real-world-http
https://github.com/budougumi0617/real-world-http

Real World HTTPの写経。一通りHTTPを叩けたので良かった。

## go-testing
https://github.com/budougumi0617/go-testing

golang.tokyoの登壇資料リポジトリ。この発表を通してだいぶtesting pkgの理解が深まった。

- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018/)

## sandbox-go-cloud
https://github.com/budougumi0617/sandbox-go-cloud

go-cloudと名前がついているが実際はWireをちょっと試した感じのリポジトリ。

## tdt-react-jest-enzyme
https://github.com/budougumi0617/tdt-react-jest-enzyme

Jestを使ってTDDするぞ！なリポジトリ。この頃になってくるとだいぶJS触るのも楽しくなっていた。VimとLSP、ALEのおかげだ。

- [Jest( >23.0.0 )、enzymeでReactのテーブル駆動テストを行う #react #test](/2018/09/28/react-table-driven-test-by-jest-enzyme/)

## RustTraining
https://github.com/budougumi0617/RustTraining

オンライン読書会[AOSN](https://aosn.ws/)でプログラミングRustの読書会が始まったので。
最近引っ越しなどで参加できていないので書けていない…

- [プログラミングRust | AOSN読書会](https://aosn.ws/workshop/15-rust/)

## waiig
https://github.com/budougumi0617/waiig

Go言語でつくるインタプリタの写経。途中までしかできていないが、この本はTDD+インクリメンタル開発で進むのでだいぶ楽しく写経ができた。

- [[書評] Go言語でつくるインタプリタ を読んだ #go](/2018/10/30/review-writing-an-gnterpreter-in-go/)

## wem10-react-sample
https://github.com/budougumi0617/wem10-react-sample

登壇用に作ったリポジトリ。StorybookなどJS周りのツールを一通り触った。次はTypeScriptでやりたい。

- [[発表資料] React+Redux 次の一歩 補足資料 #WEBエンジニア勉強会10](/2018/11/09/presentation-web-engineer-meetup-10)

## caww
https://github.com/budougumi0617/caww

Wireを使ってGoのクリーンアーキテクチャを自分で設計してみようとしたリポジトリ。
アドベントカレンダー用に用意しようとしたのだが、諸事情で頓挫した。これは他の人にも見つかってしまったリポジトリなので年明け早々にリベンジとしてちゃんと作り込みたい。


# Activity
2018年のコミット履歴。

https://github.com/budougumi0617/

## Publicなコミット =  1,286コミット
![Public only](/2018/12/glass-only-public-2018.png)
昨年が1,051コミットだったので少し増えてきた。ログを改ざんしたり、ただブログのMarkdownをコミットしているだけの日もあったが毎日コミットすることもできた。
去年と同じ通り、写経やブログのコミットが多いのは反省点だ。


## Public + Privateなコミット =  2,192コミット
![Public and Private](/2018/12/glass-all-2018.png)

昨年は1,801コミットだったので少し増えた。今年は業務でもReactを書いたり、Goを書いてコミットすることができたので、コミット内容としてはだいぶ成長できた。
来年は質も量も上げていきたい。



--- 

# まとめ(KPT)
KPTを書く前に[去年のKPTのTry](/2017/12/27/retrospective-on-github/)は以下だった。

## 去年のTry
- もっとコードでOSSに貢献する
- 自分のOSSももっと作って公開する
- 継続的なコミットは続ける
- GoとGCPでWebアプリを作る
- Kubernetesの理解を深める

k8sは少し触っていたのだが、結局公私で触れていないので全然できていない。GCPについてはSpinnakerを立ち上げたりはしていたのだがやはりちゃんと触り続けていることができなかった。

## Keep

- 継続的にコミットできた
- 他人に使ってもらえる自作OSSが公開できた
- 他人にPRしてもらえるOSSが公開できた
- 少しだが、機能変更のPRを出すこともできた
- 登壇前にサンプルリポジトリを作ることができた

## Problem
- コードでコントリビュートまだまだ足りていない
- 写経のコミットやチュートリアルやっただけのコードが多い
- 去年のTryがちゃんと拾えていなかった

## Try
- もっとコードでOSSに貢献する
- 自分のOSSを2ヶ月にひとつ公開する
- 継続的なコミットを続ける
- 3月までにGo1.11 laterとGAEでWebアプリを作る
- 8月までにKubernetesを使ってアプリを公開してみる
- 4月までにRustでCLIツールを一つ作ってみる


ちゃんと期限を切らないと目標にならないらしいので、今年はこんな感じのTryにしてみる。
来年はもっと質・量的にも濃いコミットログを作っていきたい。


# 関連
- [2017年振り返り(GitHub編)](/2017/12/27/retrospective-on-github/)
