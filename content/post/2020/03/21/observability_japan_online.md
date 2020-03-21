+++
title= "Observability Japan Online #1 に参加してo11y成熟モデルを学んだ #o11yjp"
date= 2020-03-21T10:55:46+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["report","o11y"]
tags = ["observability"]
keywords = ["observability","GCP","Stackdriver"]
twitterImage = "twittercard.png"
+++

Observabilityの勉強会に参加したので、メモ。

なお、録画されていたので、動画は以下で視聴することができる。
- https://youtu.be/mulfJV1F1nc

<!--more-->

||Observability Japan Online #1|
|---|---|
|URL| https://observability.connpass.com/event/168837/|
|会場|オンライン|
|日時|2020/03/17(火) 19:00 〜 21:00|
|ハッシュタグ|[#o11yjp](https://twitter.com/hashtag/o11yjp)|
| Toggetter| https://togetter.com/li/1482977|
|録画|https://youtu.be/mulfJV1F1nc|

# 参加理由
業務で開発を行なっていると、当然ログなどの監視ツールの利用する。  
今まで関わってきたプロダクトでも当然監視の運用がなされていたが、自分で設定したものではなかったので、「まあこういうもんなんだろうな」くらいの理解しかなかった。  
また、分散トレースなどの文脈や、ただログを吐き出すだけでなく、そこから何を分析するのか、あるいはどうすれば分析できるようになるのか、というところを知りたいところがあった。

# 参加してわかったこと
第1回だったから、「Observability（以下o11y）とは」という説明から聞くことができて、o11y初心者としては非常にありがたかった。  
また、監視とo11yの違いもなんとなくの理解だったので、知ることができてよかった。  
o11yの成熟モデルを知ることができたのもよかった。カオスエンジニアリングやSLO目標などを目指す前に、自分のプロダクトがどの段階なのか、次の段階に行くには何をすることが必要なのかわかることで、少しずつプロダクトのo11yを成長させることができそうだ。  
`pprof`の話では、継続的プロファイリングという概念を初めて知ることができた。  
日々のデプロイでパフォーマンスの変化までちゃんと見れているかというと、見れていないので、リファクタリングなり新機能を組み込んだとき、仕組みとして「パフォーマンスにどう影響があったか」を確認するのは大事だなと思った。

# 今後どう活かすのか
（業務の話をあまり書くわけにはいかないのでぼやっとして感想になるが、）最近は分散トレースについて考えることが多いので、今回聞いた内容を上手く業務の中に取り込んでいきたい。  
ちょうど個人開発でGAEを触り始めたので、Operations（旧Stackdriver）の機能を試していこうと思う。

# 以下参加メモ

## Observabilityとは
「監視」とは異なる「可観測性」の観点。監視より1段階レベルが高い。  
「動いているのか、動いていないのか」レベルが監視（Monitoring）。  
「どのように動いているか」レベルがObservability。

k8sやi18nみたいな略し方をするので、Observabilityをo11yと略すのが一般的。

## オブザーバビリティ成熟モデルについて。
[@katzchang](https://twitter.com/katzchang)

成熟モデルとして、次のように段階分けできる。

- Step.0 Getting Started: 計測を始める
- Step.1 Reactive: 受動的対応
- Step.2 Proactive: 積極的対応
- Step.3 Predictive: 予測的対応
- Step.4 Data Driven: データ駆動


### Step 0 計測を始める
まずは何を計測するのか、どう動いているのか可視化してみる。

レイテンシが正しい範囲なのか。ブラウザとかモバイルから見た正常とは？  
サービスやアプリケーションによって「正常」の定義は変わってくる。  
アラートの設定・整理をするのが監視。どのように動いているか、がo11y。  
根本原因までたどり着けるか、が監視とo11yのちがい。

例えばリクエストを受けてから、レスポンスまでの実行時間を監視していれば、アラートを鳴らすことができる。  
何か症状や課題を観測できる状態になれば、監視できていると言える。  
その症状の原因を探れるような状態に持っていくのがo11y。（実行時間中、SQLの実行部分が支配的だ、正規表現のロジックが支配的だとか）

o11yの基本的な要素はメトリック、ログ、トレース。だいたいどのサービスもこのへんを観測できるようになっている。
NewRelicはイベントも計測できる。

計測するとアラートが鳴る。

### Step 1 Reactive: 受動的対応
何かが起きたとき気づけているか、アラートに対応できるのか。DXよく障害対応できているのか。  
サービス停止、アラートへの対応ができる状態か。障害対応の改善はできているか。  
サービス停止率や障害発生率、MTTRは計測できているのか？ // これはサービス形態によって計測基準が異なる（モバイルだったらクラッシュレート）  
受動的対応がある程度できてきたら積極的対応をしていく。


### Step.2 Proactive: 積極的対応
不安定さをなくす。遅いレスポンスを直したり、パフォーマンスを良くしたり。  
普段の1.5倍パフォーマンスが悪いのがあったら、それを答えられるようにしましょう。  
安定してきているならSLAやSLOを策定できる。カイゼン活動ができる。  
たとえば、アラートは鳴らないレベルだけど、レスポンスが遅くなっているところがあってユーザーが離脱していそうだから、カイゼンしようとか。  
安定していないのに、カイゼン活動を始めたりSLOを設定したりするのは、みんな不幸になるからこの段階になるまで取り組まないほうがベター。

### Step.3 Predictive: 予測的対応
ちょうどいいスケーリングとか、コスト削減がやっとこのへんでできる。安定しているから、大胆な変更とか障害訓練ができる。  
fluentdのプロセスが落ちたときの避難訓練したいとか。  
とりあえず出してだめだったらすぐ戻す実験的デプロイとか。  
わからない部分があるのはしょうがない。ひとまず出して考えてみる。

- testing in productin, the safe way
    - https://medium.com/@copyconstruct/testing-in-production-the-safe-way-18ca102d0ef1



### Step.4 Data Driven: データ駆動

ここでやっとデプロイの評価、顧客満足の向上ができる。  
事実とフィードバックに基づいたエンジニアリング文化。  
カオスエンジニアリングのようなアプローチもできるようになる。

### o11yとは

> オブザーバビリティーチームの目標は、ログ、メトリクス、トレースの収集ではない。
> 事実とフィードバックに基づいたエンジニアリング文化の構築であり、その文化を組織全体に広げることがゴールだ。

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">The goal of an observability team is not to collect logs, metrics or traces. It is to build a culture of engineering based on facts and feedback, and then spread that culture within the broader organization.</p>&mdash; Stimulus Functions (@taotetek) <a href="https://twitter.com/taotetek/status/974989022115323904?ref_src=twsrc%5Etfw">March 17, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


## Continuous pprof
[@ymotongpoo](https://twitter.com/ymotongpoo)

Google Cloud Platform（GCP）のo11y系の機能はCloud Logging、Cloud Montoringなどがある。  
（日本語のページはまだ存在するが、）最近Stackdriverという名称はなくなり、Opeartionsという名前に統一されている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://cloud.google.com/products/operations" data-iframely-url="//cdn.iframe.ly/N50uQL6?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

o11yは分散トレース、ログ、メトリクスの三本柱で成り立つ。  
それぞれに対応しているGCPのサービスは次のようになる。

- Cloud Trace
    - https://cloud.google.com/trace
    - 分散トレースが収集できる。アプリケーションレイヤーのログ。
    - アプリケーション間の通信、関数のレイテンシーなど。
- Cloud Logging
    - https://cloud.google.com/logging
    - ログが収集できる。主に運用で必要になる。Cloud Traceより広い範囲のレイヤーをカバーできる。
    - アプリケーションログに加え、DockerやKubernetesなど、コンテナランタイムのログなどを収集できる。
- Cloud Monitoring
    - https://cloud.google.com/monitoring
    - メトリクスを収集できる。3つの中で1番収集範囲が広い。
    - ヒープサイズやディスク読み込みバイト数などの時系列データを取得することができる。
        - 例: https://cloud.google.com/monitoring/api/metrics_gcp
    - カートの中の商品数、店舗別売上高のような自分たちで定義した指標を取得することもできる。

### プロファイラ
メトリクスと異なる概念で、プロファイルを計測することもできる。

- プロファイラは決まった項目しか収集することができない。
- その代わりアプリ全体の挙動をサンプリングするため、あるパフォーマンス劣化の前後関係も確認することができる
    - メトリクスだと指定部分しか観測できないため、前後関係から確認することが難しい。
- 統計データを取得するため、サンプリング時間は10分以上かかったりする。


### pprof
Googleはプロファイルツールとして`pprof`を公開している。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/google/pprof" data-iframely-url="//cdn.iframe.ly/0XgrGZi"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`pprof`は分析と可視化のツールなので、データの収集はできない。
APIとしてprotocol buffersが公開されているので、そのフォーマットでデータを収集すると、`pprof`はどの言語でも利用することができる。

Goの場合は標準パッケージに組み込まれているので、簡単にデータの収集もできる。

- https://golang.org/pkg/runtime/pprof/
- https://golang.org/pkg/net/http/pprof/

マネージドサービスとして、Cloud Profilerも提供されている。

- https://cloud.google.com/profiler


`pprof`は継続的なプロファイルの解析にも対応している。

- https://github.com/google/pprof/blob/master/doc/README.md#comparing-profiles

`-base`オプションで比較対象のデータを指定すると、パフォーマンス向上/劣化した部分を可視化してくれる。  
これは使えばCIにプロファイラを使ったパフォーマンステストも組み込むことができる。  
Cloud Profilerを使うとフレームグラフでカイゼン具合を確認できる。

- https://cloud.google.com/profiler/docs/comparing-profiles

本番で`pprof`動かしても負荷は5%くらい。1インスタンス（コンテナ）だけ5%パフォーマンス劣化しても、それに見合うo11yが獲得できると事業判断できるなら本番で動かしても問題ないだろう。

あとからo11yを追加するのは難しいので、稼働当初からプロファイラとかAPM入れるのがおすすめ。

### 参考
- Continuous Profiling of Go programs
    - https://medium.com/google-cloud/continuous-profiling-of-go-programs-96d4416af77b

## 「Webエンジニアのための監視システム実装ガイド」発売！
[@netmarkjp](https://twitter.com/netmarkjp)

- Webエンジニアのための監視システム実装ガイド (Compass Booksシリーズ)
    - https://www.amazon.co.jp/dp/4839969817

https://docs.google.com/presentation/d/1H2JVnHbz2gG1qKedDMlBt16URsBTGmUhOFaE6GCXFrY/edit#slide=id.p

書籍「Webエンジニアのための監視システム実装ガイド」発売！
https://netmark.jp/2020/03/2020-03-03-21-21.html

上記書籍の発売経緯など。

- 「入門監視」出版による「監視」に対する熱が出版業界的にも個人的にも高まったのが出版のきっかけ。
- 口頭で毎度説明する代わりに「これ読んでください」で終わりにしたい
- 「ペットから家畜へ」
    - （あまりいい呼び方ではないのだが、キーワードになっているのであえて使っている）
    - 「x号機」「`コードネーム`」のようにサーバに愛称をつけてペットのように運用する時代ではなくなった。
    - 今は家畜のようにオートスケーリングしたり毎日のようにインスタンスを入れ替える時代。
- このような監視技術の動向などもフォローしている。


# 参考
- [testing in productin, the safe way](https://medium.com/@copyconstruct/testing-in-production-the-safe-way-18ca102d0ef1)
- [Operations | Google Cloud Platform](https://cloud.google.com/products/operations)
- https://github.com/google/pprof
- https://golang.org/pkg/runtime/pprof/
- https://golang.org/pkg/net/http/pprof/
- [Continuous Profiling of Go programs](https://medium.com/google-cloud/continuous-profiling-of-go-programs-96d4416af77b)
- [書籍「Webエンジニアのための監視システム実装ガイド」発売！](https://netmark.jp/2020/03/2020-03-03-21-21.html)


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873118646&linkId=ee34fde75fcb1ba1e17596c22e0e1db0"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4839969817&linkId=728667b4c44594f9456599c6e9c2cb06"></iframe>
