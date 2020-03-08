+++
title= "15日間勉強してAWS ソリューションアーキテクト アソシエイト試験に合格した"
date= 2019-05-12T08:30:20+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["aws"]
tags = ["aws", "saa"]
keywords = ["資格", "aws", "SAA", "ソリューションアーキテクトアソシエイト"]
twitterImage = "2019/05/12_ogp.png"
+++

GWに勉強して、AWS ソリューションアーキテクト試験に合格したのでまとめる。

![スコア](/2019/05/12_saa.png)

<!--more-->

# TL;DR
- 10連休だったのでAWSソリューションアーキテクトの勉強をした
- 試験勉強のために購入したもの
  - [AWS認定資格試験テキスト　AWS認定 ソリューションアーキテクト-アソシエイト](https://amzn.to/2Hdk6Ua)
  - [徹底攻略 AWS認定 ソリューションアーキテクト – アソシエイト教科書（通称黒本）](https://amzn.to/2VsilLL)
  - AWS公式の[オンライン模擬試験](https://aws.amazon.com/jp/certification/certification-prep/)
- `AWS Well-Architected フレームワーク`の思想と各サービスの使い方を学んでいく
- 試験に合格するだけならば、手を動かさなくても合格できる
- 今後は手を動かしてプロフェッショナルにも挑戦し、自分の設計の幅を広げていく

# なぜ受けたのか
一言でいうと、PaaSの基本的な知識がないと、アプリケーションエンジニアも最適なアプリ設計をすることができないと考えたからだ。  
私が所属するfreee株式会社ではほぼ全てのサービスがAWS上で稼働している。私はアプリケーションエンジニアで基本的にはインフラ的な部分はSREのみなさんにおまかせ状態だ。  
だが、新規設計時の初期構想や大きめの機能追加時にはインフラ面も考慮した設計をする必要がある。  
また、私以外のチームメンバーがAWSのサービスを話題にしていても「どんな機能のサービスの話をしているのか」、「何を懸念してXXXについて話しているのか」よくわからないまま聞いていた。  
そんな状態ではいけないので、最低限の知識を得るために資格勉強をしてみることにした。

## 私の試験に対する肌感
試験を始める前の私のスキルレベルは以下のような感じだった。

- 8年前くらいに応用情報技術者の資格を取得したので品質特性などの基本的な用語の意味なら今も覚えている
  - 高度情報技術者試験も何回か受けて午後2の小論文で毎回落ちるくらい
- マイクロサービスなどは勉強しているので、SPOF（単一障害点）などの用語もなんとなくわかっている
- もっと昔にCS修士は修了しているが、もう何も覚えていないも同然
- 2年ほど前からWebエンジニア。AWSを使って開発しているが、業務で自分で操作するのは構築済みのAuto Scaling GroupやS3くらい
  - EC2、ECRやECSなどの主要サービスがなんであるかくらいは知っている

こんな感じだったが、諸事情でGW前半引きこもることになっていたので久しぶりに資格勉強を始めた。

# AWS ソリューションアーキテクト アソシエイト試験について
今更必要ないかもしれないが、試験の概要を書いておく。
きちんと試験概要を全文読みたい場合は以下のリンクを読めば良い。

- [AWS 認定ソリューションアーキテクト - アソシエイト SAA-C01 試験ガイド v1.5（注 PDF）][ssa_guide]

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://aws.amazon.com/jp/certification/certified-solutions-architect-associate/" data-iframely-url="//cdn.iframe.ly/rg094Fb"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

AWSソリューションアーキテクトアソシエイト試験は、AWSのサービスを使って以下の能力を発揮できることを証明するための試験だ。

- 顧客の要件に基づき、アーキテクチャ設計原則に沿ってソリューションを定義できること
- プロジェクトのライフサイクルを通して、ベストプラクティスに基づく実装ガイダンスを組織に提供できること

[ssa_guide]:https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS%20Certified%20Solutions%20Architect%20-%20Associate%20Exam%20Guide%20001%20v1.5%20JPN.pdf

## 試験方式
2019/05/11現在のAWSソリューションアーキテクトアソシエイト試験は、テストセンターなどのPCを使って受験する。  
自由記述問題はなく、一択回答か、複数回答の選択問題をひたすら解く。

|項目|内容|
|---|---|
|問題数|65問|
|問題形式|４択 or 複数選択(選択肢5択のなかから2つ選ぶ）|
|試験時間|  130 分間|
|試験結果のスコア範囲 | 100点 - 1,000点
|最低合格スコア| 720点 |
|受験料|15,000円（税抜）

## 試験配分
問題は以下の比重で出題される。

| 分野 | 試験における比重|
|---|---|
|分野 1: 回復性の高いアーキテクチャを設計する | 34%| 
|分野 2: パフォーマンスに優れたアーキテクチャを定義する | 24%|
|分野 3: セキュアなアプリケーションおよびアーキテクチャを規定する | 26%|
|分野 4: コスト最適化アーキテクチャを設計する | 10%|
|分野 5: オペレーショナルエクセレンスを備えたアーキテクチャを定義する | 6%|
||合計 100%|

合格スコアは720点だが、（スコア範囲の情報より、）全問不正解でも100点もらえるようなので、7割弱正解すれば十分だろう。

# どうやって勉強したか
正直実際のサービスは動かさなかった。試験問題に実際の操作画面を知らないと答えられない問題はない。  
各サービスの特性を覚えて、`AWS Well-Architected フレームワーク`に則って選択肢を選べば良い。
まず`AWS Well-Architected フレームワーク`を2回ほど読んで改めてクラウドで必要な考え方を学んだ。

## AWS Well-Architected フレームワーク
`AWS Well-Architected フレームワーク`は、アーキテクトがアプリケーション向けに実装可能な、安全で高いパフォーマンス、障害耐性を備え、効率的なインフラストラクチャを構築するのをサポートする目的で開発されたフレームワークだ。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://aws.amazon.com/jp/architecture/well-architected/" data-iframely-url="//cdn.iframe.ly/l4N3lkC"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
[マイクロサービスアーキテクチャ](https://amazon.jp/dp/4873117607)のような最近のクラウドネイティブな開発に関する本を読んだり、[ソフトウェア品質特性](https://ja.wikipedia.org/wiki/ISO/IEC_9126)を知っているならばすんなり入ってくるだろう。


- [AWS Well-Architected フレームワーク 2018 年 6 月(注 PDF)][waf]

[waf]: https://d1.awsstatic.com/International/ja_JP/Whitepapers/AWS_Well-Architected_Framework_2018_JA_final.pdf

短くはないが、このPDFを読めばクラウドでサービスを構築する際に必要かつ基本として考えなければならない５つの観点を知ることができる。

- 運用の優秀性
- セキュリティ
- 可用性
- パフォーマンス効率
- コスト最適化

また、GW中ちょうどAWS Innovateが行われており、ソリューションアーキテクトアソシエイト試験受験者用の動画がいくつかあったので、そちらを一通り観た。

- [AWS 認定 - 試験対策 「ソリューションアーキテクト - アソシエイト」| AWS Innovate 2019][innovate]

もう公開は終了しているが、観なくても`AWS Well-Architected フレームワーク`を読み込めば問題ないと思う。

[innovate]:https://aws.amazon.com/jp/about-aws/events/aws-innovate/sessions/

## 本を2冊読んだ
`AWS Well-Architected フレームワーク`を読んで「AWSが考えるクラウドで実現すべき対障害性とは？コスト最適化とは？」といった思考方法がわかれば、
あとはそれを実現するためにどんなPaaS、IaaSが提供されているか知れば良い。  
AWSが提供するサービスは多岐に渡りすぎていて、全て覚えるのは無理だ。
幸いAWSソリューションアーキテクトアソシエイトは対策本は何冊もあるので、それを使って試験範囲のサービス・機能に絞って勉強した。私が購入したのは次の2冊だ。


1冊目の「AWS認定資格試験テキスト　AWS認定 ソリューションアーキテクト-アソシエイト」はおそらく一番最近出た資格本だったので、購入した。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.amazon.co.jp/AWS%25E8%25AA%258D%25E5%25AE%259A%25E8%25B3%2587%25E6%25A0%25BC%25E8%25A9%25A6%25E9%25A8%2593%25E3%2583%2586%25E3%2582%25AD%25E3%2582%25B9%25E3%2583%2588-AWS%25E8%25AA%258D%25E5%25AE%259A-%25E3%2582%25BD%25E3%2583%25AA%25E3%2583%25A5%25E3%2583%25BC%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%25B3%25E3%2582%25A2%25E3%2583%25BC%25E3%2582%25AD%25E3%2583%2586%25E3%2582%25AF%25E3%2583%2588-%25E3%2582%25A2%25E3%2582%25BD%25E3%2582%25B7%25E3%2582%25A8%25E3%2582%25A4%25E3%2583%2588-%25E4%25BD%2590%25E3%2580%2585%25E6%259C%25A8-%25E6%258B%2593%25E9%2583%258E-ebook/dp/B07R1H87Y1" data-iframely-url="//cdn.iframe.ly/6QvztGS"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

2冊目の「徹底攻略 AWS認定 ソリューションアーキテクト – アソシエイト教科書」は評判が良い通称黒本。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.amazon.co.jp/%25E5%25BE%25B9%25E5%25BA%2595%25E6%2594%25BB%25E7%2595%25A5-AWS%25E8%25AA%258D%25E5%25AE%259A-%25E3%2582%25BD%25E3%2583%25AA%25E3%2583%25A5%25E3%2583%25BC%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%25B3%25E3%2582%25A2%25E3%2583%25BC%25E3%2582%25AD%25E3%2583%2586%25E3%2582%25AF%25E3%2583%2588-%25E3%2582%25A2%25E3%2582%25BD%25E3%2582%25B7%25E3%2582%25A8%25E3%2582%25A4%25E3%2583%2588%25E6%2595%2599%25E7%25A7%2591%25E6%259B%25B8-%25E5%25BE%25B9%25E5%25BA%2595%25E6%2594%25BB%25E7%2595%25A5%25E3%2582%25B7%25E3%2583%25AA%25E3%2583%25BC%25E3%2582%25BA-ebook/dp/B07M7S9GDL" data-iframely-url="//cdn.iframe.ly/9VcljXD"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

どちらが良いというわけではないが、２つの本は章の構成が異なっているので関心事によって使い分けることができた。

- AWS認定資格試験テキストの本はAWSの各サービスごとに章がまとまっているので、「ストレージサービスについて一通り知りたい」というときに便利だった
  - 章の構成が「可用性のためにはどんなことをすればよいか？」といった疑問の調査には向いていない
- 黒本のほうは分野ごとに章がまとまっているので、「コスト最適化に関わる各サービスの機能について知りたい」というときに便利だった
  - 章の構成が「EC2に関する機能を一通りしりたい！」といった疑問の調査には向いていない

どちらも一通り読んだあと、模擬試験や練習問題で知識不足を感じたサービス、分野について通勤電車で走り読みを繰り返した。

## 模擬問題・模擬試験を受けて苦手分野の確認・復習をした
黒本の模擬問題を解き、・AWS公式の模擬試験(2,000円)を受験して知識の定着度と苦手分野の確認をした。
黒本には65問の模擬問題が付属している。また、Amazon公式では有料だが、25問の模擬試験を受けられる。
AWSの模擬試験は正否の結果をわからないので、１問１問スクショを撮ってあとで解き直した。

ブラックベルトなども見る予定だったが、諸事情で時間がとれず観なかった。

- [AWS Black Belt Online Seminar](https://www.youtube.com/playlist?list=PLzWGOASvSx6FIwIC2X1nObr1KcMCBBlqY)

# 実際に試験を受けてみて
試験自体はピアソンVUEで受けた。神奈川付近だと武蔵小杉のテストセンターがかなり日程調整しやすかった。
はるか昔に受けたSPIテストみたいな感じで淡々とマウスで選択肢をクリックしていくだけで終わった。
勉強したとおりに回答していくだけだが、強いて言うならば、模擬試験はそんなに感じなかったのだが本番の試験問題はだいぶ日本語訳が怪しいところがあった。（私はかなり苦手なほうだが、）ある程度は読めるならば英語で解いてもほうがよいかもしれない（試験中、日英両方の問題文を確認することもできる）。
試験内容的には、ストレージ関係とネットワーク関係は結構しっかり抑えないと厳しそうだった。


## 合否結果について
試験終了後にアンケートがあり、アンケートの回答を終えるとすぐ合否がわかった…らしい。私は見逃してしまった。
他の人に聞いたところ、だいぶ小さい文字で一言でるだけらしいので、一画面一画面ちゃんと読んだほうがよいようだ。
私は合否結果を見逃してしまったので、4、5時間待ってWebで結果を確認した。結果は冒頭のスクショの通り、820点で合格だった。

![スコア](/2019/05/12_saa.png)

# 終わりに
GW毎日勉強していたわけではないが、GWから始めて2週間ほどでAWSソリューションアーキテクトアソシエイトに合格することができた。
前述したとおり、今回は資格取得を最優先にしたため、実際にサービスを一切動かさなかった。なのであくまで「みんなの話に出てくるAWS用語の意味がわかるようになった」レベルだ。
試験には受かったのでブラックベルトなど見ながら手も動かしていきたい。上位資格やDeveloper資格の取得も検討しているし、もっとAWSのサービスを学び自分の設計の幅を広げていきたい。

# 参考
- [AWS 認定ソリューションアーキテクト - アソシエイト SAA-C01 試験ガイド v1.5（注 PDF）][ssa_guide]
- [AWS Well-Architected フレームワーク 2018 年 6 月(注 PDF)][waf]
- [AWS Black Belt Online Seminar](https://www.youtube.com/playlist?list=PLzWGOASvSx6FIwIC2X1nObr1KcMCBBlqY)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07R1H87Y1&linkId=700d418bd1885752f0308769ed494739"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07M7S9GDL&linkId=8166a7e16d3f94a569918026daae1e72"></iframe>
