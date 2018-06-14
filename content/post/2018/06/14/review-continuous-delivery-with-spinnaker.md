+++
title= "Continuous Delivery With Spinnakerでマイクロサービスの継続的デリバリーの課題と解決手法を学ぶ"
date= 2018-06-14T08:55:40+09:00
draft = false
slug = ""
categories = ["review", "spinnaker","kubernetes"]
tags = ["spinnaker", "book", "microservices", "k8s"]
author = "budougumi0617"
+++

Netflixが公開している無料の電子書籍をざっと読んだので紹介。  
マイクロサービスにおける継続的デリバリーのプラクティスと、継続的デリバリーのためのOSSツールであるSpinnakerの紹介がまとまっている。


**Continuous Delivery With Spinnaker**  
https://www.spinnaker.io/ebook/#continuous-delivery-with-spinnaker

[![book-cover](/2018/06/spinnaker-ebook.png)](https://www.spinnaker.io/ebook/#continuous-delivery-with-spinnaker)

# TL;DR
- 英語が得意な人ならば半日もあれば読める程度のボリューム
- マルチクラウドに展開するマイクロサービスが直面する課題やリスクが知ることができる
- Netflixが取っているアプローチやソリューションを知ることができる。
- Spinnakerというツールを使うとどのように解決できるかを知ることができる

# 本の概要
Netflixは"[マイクロサービスアーキテクチャ](http://amazon.jp/dp/4873117607)"でもAWSなどと並んで言及されている、クラウドでのマイクロサービスアーキテクチャ開発で著名な会社だ。

**Netflix Technology Blog**  
https://medium.com/@NetflixTechBlog

そのNetflixが考えるマルチクラウドへの継続的デリバリーのアプローチ・プラクティスと社内で利用されてる[Spinnaker](https://www.spinnaker.io/)の利用方法がまとまっている。

**Multi-Cloud Continuous Delivery with Spinnaker report now available.**  
https://medium.com/netflix-techblog/multi-cloud-continuous-delivery-with-spinnaker-report-now-available-6040ba83b765

電子書籍自体は無料で公開されており、英語が得意な人ならば半日もあれば読める程度のボリュームだ（自分は英語が苦手なのでKindle for MacでGoogle翻訳にコピペしながら読んでいる）。  
対象読者は以下の通りで、Spinnakerに興味がなくてもマイクサービスの継続的デリバリーを行うときの考え方を知るに大いに参考になると思う。

- なぜ継続的デリバリーを行う必要があるのか知りたい
- クラウドサービスの継続的デリバリーの手法を知りたい
- Spinnakerを使ったアプローチ

マルチクラウドサービスへの展開をする前に直面する課題やリスク、対するソリューションを先駆者から知ることができる。


# 各章の簡単なまとめ
電子書籍は11章で構成されている。各章の概要を簡単にまとめておく。

## 1. Why Continuous Delivery
なぜ継続的デリバリーが組織にとって有益なのか。

- 継続的デリバリーのメリット
  - 新機能・機能変更・バグ修正の迅速な市場投入が保証できる
  - システム化されたデプロイによって特定の時間やターゲットに対して変更を徐々に展開することができる
  - システム化されたデプロイプロセスによって障害発生時の影響範囲を抑えることができる
  - コミットとデプロイまでの時間の短縮で開発者はバグに対する解析や対応を迅速に行うことができる

安定して迅速な継続的デリバリーを実現するためには自動化を積極的に行う必要がある。

- プラクティス
  - エンジニア自身がリリースとサポートを担当できるようにすることでエンジニア自身が自分たちが作成したサービスに対して向き合うことができる。
  - 複雑なクラウド上のリソースの扱いを簡略化し、サービスの開発フォーカスすることができる。
  - ツールを統一することでひとつのチームのカイゼンを全チームが享受することができる

## 2. Cloud Deployment Considerations
クラウドに正常にデプロイするためには何に気をつければ良いか。それに対してNetflixはどのような解決策・Spinnakerを用いているのか。

- ソフトウェアのデプロイ方法やアーキテクチャはアプリケーションのスケーラビリティ（トラフィック/エンジニア/サービスの規模全てのおいての）に影響する
  - クラウドの認証情報
  - リージョンやクラウドプロパイダをまたいだトラフィック制御・ゾーン隔離
  - オートスケーリング
  - Immutable Infrastructureとデータの永続性・一貫性
  - サービスデリバリー
  - マルチクラウド
  - クラウドへのオペレーションの抽象化・簡略化
- Netflixはこれらの課題を解決するためにSpinnakerを導入している。

## 3. Managing Cloud Infrastructure
マルチクラウドに対するデプロイで発生する課題を命名規則による抽象化で解決するアプローチの紹介

- Netflixのクラウドアーキテクチャへのアプローチはサーバグループへの命名規則、Immutable Artifacts、サーバグループによって構成される。
- Netflixはred/black(blue/green)デプロイを基本としてアプリケーションを中心においたクラウドリソースの管理を行う
- マルチクラウド上の大量のリソースをインフラチームが中央集権的に管理することより、個々のアプリ開発チームによるメンテを容易にするようにしている

## 4. Structuring Deployments as Pipelines
Spinnakerというツール自体に興味があって、その概要について知りたい場合はこの章から読むと良い。

- Spinnakerによるパイプラインの構造化の概要
- Spinnakerのパイプラインは通知、展開、検証、Jenkinsのジョブなど様々なステージを組み合わせて構成することができる
- 開発チームはそれぞれでデプロイフローを最適化できるし、Spinnakerに追加される新たなステージや通知、トリガーを取り込んで継続的にデプロイフローを改善ができる。

## 5. Working with Cloud VMs: AWS EC2
- SpinnakerによってAWS EC2上にアプリケーションを展開するときの具体例
- ヘルスチェックや負荷具合をトリガーに既存のサーバーグループをオートスケーリングをする例も示されている

## 6. Kubernetes
Kubernetes(k8s)上にアプリケーションを展開する上で考慮すべきこととSpinnakerがどのように課題解決に役立つのか

- k8sのデフォルトでは実現できないConfigMap(設定情報のリソース)も含めたDockerイメージのバージョン管理
- `kubectl`からの操作では成しえないデプロイ・配置の”完了”の保証と別クラスターへの再デプロイ

## 7. Making Deployments Safer
安全のために速度を犠牲にしないデプロイをするためのプラクティスや手法。

- Blue/Green, Rollingアップデートやロールバックの基本的なデプロイ手法
- Spinnakerを利用することで実現するトラフィック制御やカナリーリリースおよびカナリーリリースの自動検証

# 8. Automated Canary Analysis
Kayentaを利用した自動カナリーリリース解析について。

**Automated Canary Analysis at Netflix with Kayenta**  
https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69

- Spinnakerを利用したカナリーリリースの自動解析
- Spinnakerを使うことで、カナリーリリースした成果物の任意のメトリクスの計測、スコア化を自動で行い、予め決めた合格点を満たすか自動判定できる
- マニュアル、アドホックな分析を減らすことで大規模・継続的なリリースを迅速かつ安全に行うことができる。

# 9. Declarative Continuous Delivery
Spinnakerで宣言的にデリバリーパイプラインを記述するをアプローチ。
GUIではなくコードでパイプラインを表現するのはは2018/06/14時点でもまだGAされていないSpinnakerの新たな取り組みだ。

**Codifying your Spinnaker Pipelines**  
https://blog.spinnaker.io/codifying-your-spinnaker-pipelines-ea8e9164998f

- 宣言的に継続的デリバリーを行う試み
- GUIによるインタラクティブなデプロイ設定が特徴的なSpinnakerでも宣言的なデプロイをサポートする動きはすでに始まっている
- 大規模な組織であるほど宣言的なアプローチは強力なソリューションになりうる

## 10. Extending Spinnaker
Spinnakerの拡張性について。（この章は継続的デリバリー一般への言及ではなくSpinnakerに特化したトピック）

- Spinnakerの機能だけでは対応できないエッジケースのために、Spinnakerを拡張することもできる
- 例えばパイプライン中にカスタムステージを作成したり、UIに独自のリンクやボタンを追加するなど。
- Typescript/Reactの知識があればフロントエンドの拡張を、Java(JVM)の知識があればバックエンドの拡張を行うことが可能

## 11. Adopting Spinnaker
Netflix内でSpinnakerがどのように広まったのか。

- Spinnakerを効果的にエンジニアリングチームへオンボーディングさせるか
- NetfilixでSpinnakerを採用することになった決め手
  - ベストプラクティスを簡単に利用できる
  - セキュアなデフォルト構成
  - マルチクラウドを標準化できる
  - デプロイフロー(既存のJenkinsジョブなど)の再利用性
  - (先駆者による)迅速なサポート体制

# 最後に
大規模サービスを展開するとき・展開したときの課題をその前に想像したり、あるいはアプローチ・解決手段を考えるのは難しい。  
そのような内容をNetflixのような先駆者が無料書籍にして公開してくれるのは感謝しかない。  
そしてそれをちゃんと読めない自分はもっと英語を学ばなければならない。まだ流し読みしかできていないので、改めてちゃんと読んでおく。

**Continuous Delivery With Spinnaker**  
https://www.spinnaker.io/ebook/#continuous-delivery-with-spinnaker

[![book-cover](/2018/06/spinnaker-ebook.png)](https://www.spinnaker.io/ebook/#continuous-delivery-with-spinnaker)

