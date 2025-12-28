+++
title= "2019年振り返り(GitHub編)"
date= 2019-12-14T08:49:49+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["esssay"]
tags = ["Retrospective", "github"]
keywords = ["振り返り", "GitHub"]
twitterImage = "twittercard.png"
+++

この記事は、[write-blog-every-week Advent Calendar 2019](https://adventar.org/calendars/3945)の14日目の記事になる。  
昨日は[@kdnakt](https://twitter.com/kdnakt)さんで「[2019年のブログをさっくり振り返る](http://kdnakt.hatenablog.com/entry/2019/12/13/233000)」だった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://adventar.org/calendars/3945" data-iframely-url="//cdn.iframe.ly/b2dtHFs"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
今年も1年の振り返りを行なうにあたり、GitHubの活動を振り返ってみた。

<!--more-->

# 2019年のコントリビュート
[作ったPR](https://github.com/pulls?utf8=%E2%9C%93&q=type%3Apr+author%3Abudougumi0617+created%3A%3E%3D2019-01-01+is%3Apublic+-user%3Abudougumi0617+is%3Aclosed+)はこんな感じだった。  
昨年までは誤字修正のPRが多かったが、今年は少しバグフィックス的なPRを作ったりできた。業務で使っていた`ory/fosite`や`Versent/saml2aws`のメンテPRを作れたのもよかった。

- nazonohito51/oop-study-payroll-system
  - https://github.com/nazonohito51/oop-study-payroll-system/pull/20
- OWASP/Go-SCP
  - https://github.com/OWASP/Go-SCP/pull/67
- ohke/sqs-polling
  - https://github.com/ohke/sqs-polling/pull/1
- Versent/saml2aws
  - https://github.com/Versent/saml2aws/pull/356
- census-instrumentation/opencensus-website
    - https://github.com/census-instrumentation/opencensus-website/pull/716
- yoshitaku-jp/api-wbew
    - https://github.com/yoshitaku-jp/api-wbew/pull/1
- ory/fosite
    - https://github.com/ory/fosite/pull/360
- t0w4/go_tsumamigui_book_codes
    - https://github.com/t0w4/go_tsumamigui_book_codes/pull/3
- write-blog-every-week/write-blog-every-week-remind
    - https://github.com/write-blog-every-week/write-blog-every-week-remind/pull/33
    - https://github.com/write-blog-every-week/write-blog-every-week-remind/pull/23
    - https://github.com/write-blog-every-week/write-blog-every-week-remind/pull/28
    - [いつもお世話になっているslackチーム](https://kojirooooocks.hatenablog.com/entry/2018/09/21/223841)で使われているLambdaのコードの修正
- GoCon/2019-Autumn
    - https://github.com/GoCon/2019-Autumn/pulls?q=is%3Apr+is%3Aclosed+author%3Abudougumi0617

あと、はじめてGo本体のリポジトリにissueを書いた。来年はPRもつくりたい。
- https://github.com/golang/go/issues/34650

# OSS
今年は3つOSSを作った。他のひとに使ってもらったり、自分のプロダクトに導入できているので作って良かったなと思う。
それなりにスターを貰えるOSSになったのも自信につながった。

## blog-kpi-collector
- https://github.com/budougumi0617/blog-kpi-collector
  - `clasp`コマンドを使ってKPIを取得するgoogleスプレッドシートを生成するスターターキット
- [GoogleスプレットシートでWebサイトのKPIを集計するためのOSSを作った #blog #gas](/2019/01/26/publish-blog-kpi-collector/)

## layer
- https://github.com/budougumi0617/layer
    - Analyzer: Checks whether there are dependencies that illegal cross-border the layer structure.
- [[Go] レイヤードアーキテクチャの階層構造を守らないimportを警告するlinterを作った](/2019/10/18/launch-layer-for-the-layered-achitectures/)


## dkl
- https://github.com/budougumi0617/dkl
    - Shows containers(or pods), and exec(login) selected it.
- [[dkl] 実行中のDocker ContainerやKubernetes Podを一覧して選択、アタッチするツールを作った](/2019/09/28/release-dkl/)


# Dotfile
https://github.com/budougumi0617/dotfiles

 - 2019年の差分
  - https://github.com/budougumi0617/dotfiles/compare/2cffc3...3e7ee99


zshの起動があまりに遅かったので、lazy loadをするようになった。
また、Vimでgoplsを使うようになった。ただ、まだvim-go経由で呼んでいるだけなので、使いこなせてはいない。

# その他今年作ったリポジトリ
[created:>=2019-01-01 is:publicで絞った結果](https://github.com/budougumi0617?utf8=%E2%9C%93&tab=repositories&q=created%3A%3E%3D2019-01-01+is%3Apublic&type=&language=)

## til
- https://github.com/budougumi0617/til

今更ながらTIL（Today I Learned）リポジトリを作った。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fsyossan.hateblo.jp%2Fentry%2F2016%2F02%2F16%2F144305" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

簡単な動作検証だったり、新しく覚えたり「よさそう」と思ったコード断片を雑に突っ込んでいる。
Pythonディレクトリが生えたり、自分の技術の広がりを感じられてよい。
「CI環境の作り方も覚えたい/CDの動作確認をしたい」というときは別リポジトリを作っている（存在を忘れるのでこのリポジトリにsubmoduleしている。）。

## lambda-go-dynamodb-local
- https://github.com/budougumi0617/lambda-go-dynamodb-local

CircleCIでDynamoDBを操作するテストを試した。

- [DynamoDBを操作するコードをamazon/dynamodb-localコンテナとCircleCIを使ってCIする #aws](/2019/06/02/use-dynamodb-local-on-circleci/)

## sandbox-cloud-functions
- https://github.com/budougumi0617/sandbox-cloud-functions

Cloud FunctionsがGoをサポートしたので触ってみた。

- [BetaリリースされたGoのGoogle Cloud Functionsを試す #gcp #golangjp](/2019/01/17/try-go-on-google-cloud-functions/)

## mysql-sakila
- https://github.com/budougumi0617/mysql-sakila

SQLの練習用にDockerイメージを作った。
`JOIN`などの素振りをしたいけど、複数テーブルにそこそこデータ入っているデータベースないかな？というときに使う。

- [クエリの練習用に20以上のテーブルと大量データを事前投入したMySQLイメージをDocker Hubで公開する](/2019/03/20/publish-docker-image-for-sql-training/)

## go-sql-sample
- https://github.com/budougumi0617/go-sql-sample

技術書典6に執筆した原稿のサンプルコード。
[改訂2版 みんなのGo言語](https://www.amazon.co.jp/dp/B07VPSXF6N)が出るまで`database/sql`パッケージの使い方について触れた書籍がなかったので。

- [golang.tokyoの技術書典6新刊「文Go」に「Goにおけるデータベース実践入門を寄稿しました。 #技術書典](/2019/04/11/shoten6/)


## cig
- https://github.com/budougumi0617/cig

「Go言語による並行処理」でちょっと写経をするとき用のリポジトリ。
ただ、結局TILリポジトリで書くようになったのであまりコードは入っていない。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873118468&linkId=3440f4751eb741a94f023b71e9b4e707"></iframe>

## gas-typescript
- https://github.com/budougumi0617/gas-typescript

GASをGitHubで管理したいなと思っていたら、TypeScriptに対応していたのでためした。

- [Google Apps ScriptをTypeScriptで実装する(clasp/TSLint/Prettier) #gas #typescript](/2019/01/16/develop-google-apps-script-by-typescript/)

## gomodules-explore
- https://github.com/budougumi0617/gomodules-explore

Go 1.12 Release Partyで発表するためにGo Modulesの挙動を比較した。

- [Go Modulesの概要とGo1.12に含まれるModulesに関する変更 #golangjp #go112party](/2019/02/15/go-modules-on-go112/)

## claat-sample
- https://github.com/budougumi0617/claat-sample

ハンズオンを作るならば`claat`が最高だと思う。`github.io`で公開するためのリポジトリ。

- [claatを使ってCodelabs形式のオリジナルハンズオンを作る](/2019/03/27/create-codelab-page-by-claat/)

## homebrew-tap
- https://github.com/budougumi0617/homebrew-tap

いままでずっと`brew tap`用のリポジトリは配布したいツールごとに毎回つくらないといけないと勘違いしていた。
実はひとつのリポジトリですべて管理できるらしいので、このリポジトリで統一することにした。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://docs.brew.sh/Taps" data-iframely-url="//cdn.iframe.ly/ahnaDEK"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## review-uploader
- https://github.com/budougumi0617/review-uploader

`Re:VIEW`からPDFをCircleCIでビルドして、google driveにアップロードするためのカスタムコンテナイメージ。

- https://hub.docker.com/repository/docker/budougumi0617/review-uploader

## pronami_php
- https://github.com/budougumi0617/pronami_php

PHPを勉強する上で「気づけばプロ並みPHP改訂版」という書籍を写経したリポジトリ。

- [[書評] 気づけばプロ並みPHP改訂版を読んでPHPに入門した](/2019/11/25/review_pronami_php/)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4865940650&linkId=3976f67b396e3ae98803b720c6d6a632"></iframe>

# Activity
2019年のコミット履歴。

https://github.com/budougumi0617/

## Publicなコミット =  2,016コミット
![Public only](/2019/12/glass-only-public-2019.png)
ブログのコミットも多いが、今年は昨年より大幅にコミット数が増やすことができた。
増加としては以下の要因が大きいと思われる。

- ブログのサンプルコードをたくさん書いた
- 人に公開できるようなOSSをいくつか書いた
- Go Conference 2019 AutumnのWebサイトのメインメンテをしていた

昨年はややズルもしながら毎日コミットをしていたのだが、今年は海外旅行でコーディングから離れていた時期もあり、毎日コミットは辞めた。

- 2017年 1,051コミット
- 2018年 1,286コミット


## Public + Privateなコミット =  2,956コミット
![Public and Private](/2019/12/glass-all-2019.png)

privateなコミットだけを見ると昨年とあまり変わらず800程度のコミットのようだ。
10月はまるまる有給消化で休んでいたので、それを考えると営業日あたりのprivateコミット数は昨年より増えていそうだ。
今年はTerraformやHelmのコミットをすることができた。また、転職してPHPやPythonも書くようになった。
来年はフロントのタスクもひろってVue.jsもすることになるので、Go以外でも事業に貢献できるようになりたい。

- 2017年 1,801
- 2018年 2,192

まだ年末まで2週間ほどあり、OSSもあと1つ作るので、3,000コミットはいけそうだ。

--- 

# まとめ（KPT）
KPTを書く前に[去年のKPTのTry](/2018/12/15/retrospective-on-github/)は以下だった。

## 去年のTry
- もっとコードでOSSに貢献する
- 自分のOSSを2か月にひとつ公開する
- 継続的なコミットを続ける
- 3月までにGo1.11 laterとGAEでWebアプリを作る
- 8月までにKubernetesを使ってアプリを公開してみる
- 4月までにRustでCLIツールを1つ作ってみる

継続的に振り返っていなかったのでほとんど達成できていないという衝撃的な結果になってしまった。  
正直この記事を書くために見返して、「あ、こんな目標たててたのか…」と衝撃を受けたレベルで忘れていたので、来年は定期的に振り返って進捗を確認する。


## Keep

- 単純なコミット数を増やすことができた
- 2桁Starをもらった自作OSSが公開できた
- ブログを書くだけでなく、検証用のコードをちゃんと作ることができた
- Terrafrom、Helm、Python、PHPなどの新しい挑戦も始められた
- 普段使っているOSSにPRを出すことができた
- golang/goにissue reportができた

## Problem
- feature PRをOSSに出せていない
- 去年のTryを全く達成できていなかった
- privateコミットの量がぜんぜん増えていない

Tryが全く達成できていないなかったのはどうしようもない。
また、今年もあまり機能追加のPRをOSSに対して出すことが出来なかった。


## Try
- 業務で貢献してprivateコミットを増やす
- 3月までにGAEで家族用のLINE Botを作る
- 半年に１つ以上2桁StarがもらえるようなOSSを公開する
- Vue.jsとFirebaseを使ってフロントエンドを構築してみる
- 定期的にTryの進捗を確認する
- OSSに対して機能追加のPRを6個以上出す

来年のTryは毎月振り返って進捗を確認しようと思う。ひとまずSlackにリマインダを仕込んでおいた。  
直近としてはLINEをインターフェースにGoogleスプレットシートなどにデータ入力をしたいので、そのためのサービスを作ろうと思う。  
また、来年はフロント力を上げたいのでVue.jsも個人的に触ってみる。


# 終わりに
あまり成果が出ていないと思っていた1年だったが、振り返ってみるといろいろ挑戦していたのがわかってよかった。  
昨年までは「全然だめだ…」という気持ちばかりだったが、今年は「自分にしてはそこそこアウトプットできているかな？」と思えた。  
とはいえ他の人に比べるとまだまだなので、来年はもっと成果を出していきたい。

[write-blog-every-week Advent Calendar 2019](https://adventar.org/calendars/3945)の15日目は[@kz_morita](https://twitter.com/kz_morita)さん！

# 関連
- [2017年振り返り(GitHub編)](/2017/12/27/retrospective-on-github/)
- [2018年振り返り(GitHub編)](/2018/12/15/retrospective-on-github/)

