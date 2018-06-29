+++
title = "Spinnakerを使ったカナリーリリースの自動評価 #spinnaker #kayenta"
date = 2018-06-29T08:33:06+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["spinnaker", "gke", "kubernetes"]
tags = ["spinnaker", "gke", "kayenta", "k8s", "gcp"]
+++
Netflixが4月に発表した[Spinnaker](https://www.spinnaker.io/)と[Kayenta](https://github.com/spinnaker/kayenta)を使ったカナリーリリースの自動分析を試してみた。

- Automated Canary Analysis at Netflix with Kayenta
  - https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69

<!--more-->

CDツールであるSpinnakerを使えば、Kubernetes(k8s)へのカナリーリリースだけでなく、カナリーリリースを自動評価して新バージョンの良し悪しを判断、その後の本番でのスケールアウトまで自動で行うことができる。  
今回は以下のブログのサンプルリポジトリと設定を使って試してみる。k8sとモニタリング環境はGKEとStackdriverを使う構成になる。

**AUTOMATED CANARY ANALYSIS USING SPINNAKER - CODELAB**  
https://ordina-jworks.github.io/cloud/2018/06/01/Automated-Canary-Analysis-using-Spinnaker.html

# TL;DR
- Spinnakerを使えばk8s上にカナリーリリースを簡単に行うことが出来る
- カナリーリリースをしたあとはそのまま自動でカナリーリリースを分析・評価できる
- 評価結果に応じて本番デプロイを継続したりロールバックすることができる
- 評価項目はDatadog、Prometheus、Stackdriverなどで取れるメトリクスならなんでもOK。独自指標も使える

![Execute auto canaly analysis](/2018/06/30_execute-auto-canary-analysis.png)

# カナリーリリースとSpinnaker
Webサービスのリリース手法の一つに[カナリーリリース（カナリアリリース）](https://martinfowler.com/bliki/CanaryRelease.html)が存在する。  
カナリーリリースは新旧2つのバージョンを同時に本番に稼働あせ、問題が無いことを確認してリリースする手段だ。  
ここで「新バージョンに問題がないこと」とはどう判断するのだろうか？メトリクス情報やアラート情報を毎回一定時間確認すればよいのだろうか？  
また、問題の有無を確認したあとはどうしたらよいのだろうか？問題がなかった場合はスケールアウトを始めればよいのだろうか？問題があった場合は旧バージョンへのロールバックを始めればよいのだろうか？

自動デプロイを行うCIツールは多いが、カナリーリリースを実施したあとまで自動化できるのがNetflixのSpinnakerとKayentaだ。

- Automated Canary Analysis at Netflix with Kayenta
  - https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69

- Spinnaker
  - https://www.spinnaker.io/
- Kayenta
  - https://github.com/spinnaker/kayenta

# 今回試した動作環境
Spinnakerやk8sの環境は以下の通り。

- GKE(Kubernetes)クラスターバージョン
  - 1.10.4-gke.2
- Spinnaker
  - 1.8.0

2018/06/28現在Spinnakerを起動しただけではカナリーリリースの自動評価を行うことはできない(Kayentaが有効にならない)。  
Kayentaの有効化も含めたSpinnakerの構築手順は別の記事にまとめている。

- [GKE上でStackdriverと連携したSpinnaker/Kayenta環境を構築する #spinnaker #kayenta](/2018/06/23/how-to-start-auto-canary-analysis/)

# 今回用意したデプロイパイプライン
今回はデプロイ用のデモアプリリポジトリをGitHubに用意しているブログを参考に作成したパイプラインは3種類。  
また、Spinnakerとは別に、GitHubリポジトリが更新されたときにGCRのビルドトリガーが走り、gcr.io上にコンテナが作成されるようになっている。

- GCRに新しいコンテナイメージがアップロードされたのを確認すると、staging相当のクラスターにコンテナをデプロイするパイプライン
  - 今回はそのままデプロイしているが、通常はテストなどのステージを挟むだろう
![Deploy to Staging](/2018/06/30_deploy-to-staging.png)
- staging向けのパイプラインが完了したことをトリガーにカナリーリリースを行うパイプライン
![Deploy to Prod](/2018/06/30_deploy-to-prod.png)
- カナリーリリース用のパイプラインが完了後お片付けをするパイプライン
![Clean up](/2018/06/30_clean-clusters.png)

カナリーリリースを行うパイプラインは以下のようなステージを持つ。

- stagingのコンテナを用いてCanaryリリースをデプロイするステージ
- productionのコンテナ(現バージョン)のコンテナを用いて比較対象用のbaselineリリースをデプロイするステージ
- カナリー分析用の設定を用いてメトリクスを計測して上記の2つのリリースを比較する自動分析ステージ
- 分析結果が良好ならばstagingのコンテナを改めて本番クラスターにデプロイするステージ

すべてのデプロイは本番用のロードバランサを使うようにしてある。
通常のデプロイと変わらないので、カナリーリリースの割合も指定できる

# カナリーリリースの分析設定
カナリーリリースの分析でモニタリングできる任意の項目を評価対象に指定することができる。
今回の場合はStackdriver経由のモニタリング結果を用いて自動評価をするため、Stackdriverで取得できる任意のメトリクス情報を指定可能だ。
StackdriverにもAWS(CloudWatch?)、Datadog、Prometheusに対応している。

- Set up canary support
  - https://www.spinnaker.io/setup/canary/

![Choose metrics](/2018/06/30_choose-analysys-data.png)

Stackdriver loggingで予めログの指標を作成しておけば、ユーザー独自の評価項目を作成することもできる。
![Define metrics](/2018/06/30_define-metrics.png)
![Choose original metrics](/2018/06/30_use-stackdriver-setting.png)

複数の評価対象を指定して、各項目の結果について重み付けすることもできる。

今回はブログに習い、CPU負荷とメモリ負荷、Stackdriverで作成したログ指標を用いたメトリクス分析設定を作成してある。


# カナリーリリースの自動分析
この状態でパイプラインが開始され、baselineクラスターとCanaryクラスターにデプロイが完了すると、分析設定に基づいたカナリリリースの自動分析が始まる。  
下記の画像は自動分析をしている途中のパイプラインだ。5分おきにメトリクスを分析して1時間カナリーリリースのパフォーマンスを検証している。

![Execute auto canaly analysis](/2018/06/30_execute-auto-canary-analysis.png)

分析設定で指定した一定以上のスコアが出ていれば分析ステージは完了し次のステージに進むことができる。  
自動評価中に取得したメトリクスデータはグラフ化されて確認することができる。

![Check report](/2018/06/30_check-report.png)

# 所感
自分は普段業務ではアプリ層のことしかやっていないため、Kubernetesすらちゃんと動かせるのか怪しい。  
そんな自分にでも継続的デリバリーからカナリーリリース、その分析まで自動化することができた。  
ただ、2,3日動かしたままにしていたところ、GCRからコンテナのTagが取得されなくなったりした。  
そのときはPodをリスタートすれば良いとのこと（自分は`hal deploy apply`を再度実行してなおした）。

- Continuous DeliveryツールのSpinnakerを動かしてみた
  - http://bufferings.hatenablog.com/entry/2018/02/17/124648

# 終わりに
Spiinakerのパイプライン設定もあまりわかっていなかったので今回参考にした[ブログ記事](https://ordina-jworks.github.io/cloud/2018/06/01/Automated-Canary-Analysis-using-Spinnaker.html)はかなり勉強になった。具体的な設定方法などが丁寧に書いてあるので手元で動かしてみたいときはぜひ参考にしてほしい。  
(Infrastructure as Code的に))GUIでぽちぽちするだけで設定が変更できてしまうところに懸念を抱く人もいるかもしれないが、現在パイプラインの設定をYAMLのみで編集するプロジェクトも進行中である。

- Codifying your Spinnaker Pipelines
  - https://blog.spinnaker.io/codifying-your-spinnaker-pipelines-ea8e9164998f
- pipeline-templates
  - https://github.com/spinnaker/pipeline-templates

# 参考
- [AUTOMATED CANARY ANALYSIS USING SPINNAKER - CODELAB](https://ordina-jworks.github.io/cloud/2018/06/01/Automated-Canary-Analysis-using-Spinnaker.html)
- [Automated Canary Analysis at Netflix with Kayenta](https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69)
- [Set up canary support](https://www.spinnaker.io/setup/canary/)
- [Continuous DeliveryツールのSpinnakerを動かしてみた](http://bufferings.hatenablog.com/entry/2018/02/17/124648)

# 関連
- [GKE上でStackdriverと連携したSpinnaker/Kayenta環境を構築する #spinnaker #kayenta](/2018/06/23/how-to-start-auto-canary-analysis/)
- [Continuous Delivery With Spinnakerでマイクロサービスの継続的デリバリーの課題と解決手法を学ぶ](/2018/06/14/review-continuous-delivery-with-spinnaker/)
- [[発表資料]Spinnaker入門 #cndjp5](/2018/04/27/spinnaker-introduction/)
- [GKEでSpinnakerを使ったKubernetesの継続的デリバリーを試してみた](/2018/04/24/spinnaker-with-google-kubernetes-engine/)


