+++
title= "GCPUG Tokyo Container Builder Day February 2018 #gcpug 参加メモ"
date= 2018-02-05T19:08:24+09:00
draft = false
slug = ""
categories = ["report", "gcp"]
tags = ["gcb", "gcr","gsr","gcpug"]
author = "budougumi0617"
+++

今年はGoogle Cloud Platform(GCP)を使ってみようと思っているので、[GCPUG](https://gcpug.jp/)の勉強会に参加してきた。  
02/05はGoogle Container Builder(GCB)の話だった。


|||
|---|---|
|URL|https://gcpug-tokyo.connpass.com/event/75961/|
|会場|六本木ヒルズ 森タワー 18F メルカリ|
|日時|2018/02/05(月) 19:00 〜 21:30|
|ハッシュタグ| [#gcpug](https://twitter.com/search?q=%23gcpug) |

# TL;DR
- Google Cloud BuilderはシンプルなコンテナーイメージをビルドするGCPのサービス
  - https://cloud.google.com/container-builder/?hl=ja
- １日120分も無料でビルドできる
- シンプルにコンテナをビルドするだけ。GCP使っていなかったり、テストや複雑なビルドをしたいならば他のCIのほうが良いかも
- ビルドの1ステップ1ステップをステップごとに異なるビルド用のコンテナの上で`docker run`していることを理解してないとハマる
  - https://github.com/GoogleCloudPlatform/cloud-builders
- GCPのドキュメントは基本的にAPIリファレンスを読むとわかりやすい。人間用のドキュメントを読んでいてもあまりわからない。
- GCPUGのGCPノウハウは以下のリポジトリにある
  - https://github.com/gcpug/nouhau
- もっと話したい人はGCPUG Slack #g-tools_jaチャンネルで
  - https://gcpug.jp/join

# Cloud Tools Introduction by sinmetal
https://goo.gl/6WVrR5  
[@sinmetal](https://twitter.com/sinmetal)

GCPのサービスの中でToolカテゴリは４つある。その中の2つの話

- [Google Cntainer Registry(GCR)](https://cloud.google.com/container-registry/?hl=ja)
- [Google Source Repositories](https://cloud.google.com/source-repositories/?hl=ja)

## Google Container Builder

- Build Triggerと併用すれば簡単なCIも出来る。Container Registoryに成果物のイメージとビルドログがでてくる。
  - 他のCIのようにビルドするブランチを指定したり、ブランチ別のビルドも設定出来る。
- 毎日120分無料でビルドできる。
- ビルドソースはSource Resistories、GitHub、GitBucketから取得、もしくはローカルからtarにしてプッシュできる（GitLabは非対応）
- ビルドステップはDockerfile, cloud.yamlで定義する
- **1ステップ1ステップを毎回新しいコンテナで実行するのが特徴**
- ツールは公式で用意されているコンテナイメージで使ったほうがよい。
  - https://github.com/GoogleCloudPlatform/cloud-builders
- `gcloud bulds submit`コマンドでローカルから実行できる
  - [Container Builder の概要 - gcloud コマンドライン ツールの使用](https://cloud.google.com/container-builder/docs/concepts/overview#using_the_gcloud_command-line_tool)
- **fビルド行為をローカルで実行して動作確認することもできる**
  - [Container Builder の概要 - ローカルでのビルドの実行](https://cloud.google.com/container-builder/docs/concepts/overview#running_builds_locally)
- 認証情報などをCloud KMSと連携して扱えるのも魅力
- サードツールへの通知やビルドステータスによるマージブロックなどしたかったらPub/Sub、Functionsなどと組み合わせて自作する必要がある
  - （Slack通知についてはLTの方が触れていたので後述）


## Google Cntainer Registr(GCR)
- GCRは非公開のDockerレジストリ。
- GCPのIAMと連携してコンテナイメージを管理できる
- 実際に作成されたコンテナイメージはストレージの中にある

## Google Source Resistories
- Private Git Repository。それ以上でも以下でもない。
- Issue管理やPull Request機能といった機能はない
- Cloud Debuggerと連携することができる
- GitHub レポジトリや Bitbucket レポジトリとも連携できる

## 終わりに
この辺の話をしたいときは[GCPUG Slack](https://gcpug.jp/join) #g-tools_jaで！

GCBに限らずGCPUGのノウハウはこのリポジトリで共有されている  
https://github.com/gcpug/nouhau

# Google Container Builder と友だちになるまで
https://www.slideshare.net/lestrrat/google-container-builder-87244724  
[@lestrrat](https://twitter.com/lestrrat)

- GCBは基本的にGoogle Container Resistoryにコンテナを突っ込むためにビルドするサービス
- 以下の３つの理由がないと使わないほうが良い。ちゃんとテストを流したいなどがあるなら別のCIを使ったほうがよい
  - コンテナイメージをビルドしたい（だけ）
  - GCP上にリソースが全部ある
  - もうリモートデバッグしたくない


## GitlabにあるmonorepoをGCBでgcr.ioにデプロイするときの知見
- GitlabにあるmonorepoをGCBでビルドすることに。テストは別で行っていて、デプロイ先はgcr.ioだった
- (GitLabは連携できないので)ローカルで`google cloud builds`コマンドをすると指定したフォルダーがtarになって送信される。（最初のステップに必要なものだけあればよい）
  - 全部ストレージに保存されるからデカイmonorepoあげるとヤバい。
- `cloud.yaml`を使ってビルドする。GCBのビルドは以下のイメージ

```go
for step in config.steps
  docker run step.image step.args
done
```

- ビルドで使うイメージは基本的に公式イメージで。ただし1コマンドだけ実行しているようでもラッパースクリプトが動いているので注意
- 各ステップで情報を引き回したいときは`Volume`でコンテナにマウントした中に保存しておく
- デバッグしたいときは`entorypoint`でbashを動かす。APIリファレンスまで見に行かないと載ってない
  - https://cloud.google.com/container-builder/docs/api/reference/rest/v1/projects.builds#buildstep
- どうしてもひとつのステップで複数のコマンドを使いたいときは自作コンテナでビルドする
  - gcloud、helm、kubectl...なんてことは[公式ビルド用コンテナ](https://github.com/GoogleCloudPlatform/cloud-builders)ではできない
  - gcr.ioに上げておけば自作コンテナイメージでビルドもできる
- 困ったらローカルで実行してみるのが一番。クラウド上と同じ挙動をしてくれた
  - https://github.com/GoogleCloudPlatform/container-builder-local
- googleのドキュメントは基本的にAPIリファレンスを読むとわかりやすい。人間用のドキュメントを読んでいてもあまりわからない。

# LT
## GitLab Runnerの話

[@tn961ir(tnir)](https://twitter.com/tn961ir)

- [GitLab Runner](https://docs.gitlab.com/runner/)
- GitLabもGoogle Cloud Resitoryを使っていたが今は[GitLab Container Registry](https://docs.gitlab.com/ce/user/project/container_registry.html)
- どちらもベンダーロックインを避けられる。スレーブを用意すればマルチOSでビルドできるのもGitLab Runnerの魅力
- GCE上で実行した場合はStack Driverとも連携出来る。
  - https://gitlab.com/rweda/stackdriver-gitlab-ci
- GitLab実践ガイドも最近発売されたとのこと
  - https://book.impress.co.jp/books/1116101163


---

## Google Cloud BuilderとSlack連携の話
[@yoshikai_](https://twitter.com/yoshikai_)

- GCBのStatusをSlack通知
- GCBのビルドステータスが変化したときに通知がいく。Slack以外はGCPのサービスで実現できる
- GCB -> Pub/Sub -> Funstionsという流れでSlackに通知する。それぞれが有効になっていればGCPのWeb UIから設定可能
- 詳細は公式チュートリアルがある
  - [サードパーティサービスの通知の設定](https://cloud.google.com/container-builder/docs/tutorials/configuring-third-party-notifications?hl=ja)
- 公式チュートリアルのFuctionsのままだと`build.id`だけで何の結果だかわからないのでもう少し作り込む必要があり。
  - [Tweet](https://twitter.com/apstndb/status/960487534500356096)によると、この辺叩けば出来るんじゃないかとのこと [Method: projects.builds.get](https://cloud.google.com/container-builder/docs/api/reference/rest/v1/projects.builds/get?hl=en)

google assistantアプリもリリースしているらしい。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">えいえい おこった？💕 <a href="https://twitter.com/hashtag/gcpug?src=hash&amp;ref_src=twsrc%5Etfw">#gcpug</a> <a href="https://t.co/UkbhWpZMYe">pic.twitter.com/UkbhWpZMYe</a></p>&mdash; みゅず (@mu248321) <a href="https://twitter.com/mu248321/status/960488259565465600?ref_src=twsrc%5Etfw">2018年2月5日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


**怒ってないよ - Google Assitant アプリ**  
https://assistant.google.com/services/a/uid/0000001a6f41b0be?hl=ja

クソアプリって名前はリジェクトくらうとのこと。

# 参加して
GCPUG主催イベント初参加だった。悪い言い方をするとGCBは他のCIツールと比べて機能が少ないようだ。  
ただ、コンテナイメージのビルドするのにステップごとに新しいコンテナを利用するという仕組みはすごい新鮮で驚きだった。  
ビルド自体をステップごとにコンポーネント化するという考えは複雑化や属人化を避けることも可能なのかな？  
GKEの上でGoのアプリでも動かしたいなと思っているので、ある程度使いこなせてきたらデプロイ用のコンテナ生成をGCBでやってみるのもいいのかもしれない。  
GCPは結構ドキュメントの日本語がちゃんとされていて、これから動かし始めようと思っている英語が不得意の自分的には（パット見）結構安心していたのだが、けっこう大事なところが書かれていなかったりするらしい。  
困ったらAPIリファレンスまで読んでみる、という本筋以外の知見も得られて良かった。  
（某窓の会社のリファレンスサイトもサンプルリポジトリのコードみないと使い物にならないので、個人的にはまあそんなものなのかなとも思っている）

Circle CI2.0もローカルでビルドを実行して試せるが、クラウド上の挙動を再現できていないこともあると会社の人が言っていた気がする。  
（自分も使っているが、`go test`してCodecovに結果流すくらいはローカルで出来る）

**Using the CircleCI Command Line Interface (CLI)**  
https://circleci.com/docs/2.0/local-jobs/


他にも以下のカンファレンスついても案内があった。

**JAPAN CONTAINER DAYS V18.04**  
2018.04.19(Thu)  
http://containerdays.jp/

**builderscon tokyo 2018**  
2018年9月6日(木), 2018年9月7日(金), 2018年9月8日(土)  
http://2018.tokyo.builderscon.io/



