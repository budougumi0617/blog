+++
title= "2020年振り返り"
date= 2020-12-27T07:23:52+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["esssay"]
tags = ["Retrospective"]
keywords = ["振り返り"]
twitterImage = "twittercard.png"
+++

2020年を振り返ってみる。GitHubベースの振り返りは別記事にまとめた。

- [2020年振り返り(GitHub編)](/2020/12/15/retrospective-on-github-2020/)

この記事はwrite-blog-every-week Advent Calendar 2029の25日目の記事となる（2日遅れ）。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://adventar.org/calendars/5530/embed" data-iframely-url="//cdn.iframe.ly/CFIQIEi"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# 主な出来事

## 双子が産まれた
コロナ禍もそうなのだが、6月に双子が生まれて生活が一変した。  
育休を4か月取った後、11月から働いている。  
プライベートなことなので具体的なことは書かないが、双子の育児はけっこうけっこう大変である。

## ブログなどのKPI
昨年から毎週Twitterのフォロワー数とブログのスコアを集計している[^kpi_issue]。

[^kpi_issue]: 今twitterのフォロワー数がうまく取れていないので直さないと…

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/blog-kpi-collector" data-iframely-url="//cdn.iframe.ly/13K41P4?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

今年は以下のような数値だった（6月のtwitterフォロワー数は取得に失敗している）。

|集計日|Twitter フォロワー数|ブログの週間PV|ブログの累計はてブ数|
|---|---|---|---|
|2019/12/29|752|5395|2277
|2020/03/29|868|3237|2355
|2020/06/28|-1|3018|2421
|2020/09/27|973|1917|2498
|2020/12/25|1066|2152|2737

いつの間にかフォロワー数が1,000超えていた。少しでも役に立ってればいいなと思う。

2020年のブログは42記事執筆した。  
年間PV数は2020/12/26現在で155,400PVくらいだった、月間平均にすると約13,000PVだが、ゆるやかにPVが下がり続けているので来年は頑張りたい。
はてブ数が多かったのは以下。

- [Goらしさとは何なのか考える](/2020/10/23/think_go_way/)
- [[Go]次世代イメージcimg/goとcircleci/go Orbsを使った2020年版CircleCIの環境構築](/2020/06/08/circleci_cimg_go_2020/)

CircleCIの話は全然他に記事がないのでよいものが書けたと思う。

また、会社のブログにも2記事寄稿した。
<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fautomatic-release-on-github-actions" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fgo-code-reading" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

## PHPer Kaigiで登壇した
コロナ禍前なので1年経っていないのが信じられないが、PHPer Kaigi2020で登壇した。  
Go以外のイベントで登壇できたのはよかった。

<script async class="speakerdeck-embed" data-id="aab8bb3c355b4fc4b03cad50aec2b72c" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

## 技術書典応援祭、技術書典9で技術同人誌を販売した
golang.tokyoで技術書典に参加し、今年は2回寄稿した。

- [[冒頭公開]技術書典 応援祭にgolang.tokyo新刊「Gopherの休日2020冬」で参加します #技術書典](/2020/02/28/shote8-golangtokyo/)
- [#golangtokyo 技術書典9の新刊に「LeetCodeでアルゴリズムとデータ構造エクササイズ」を寄稿しました #技術書典](/2020/09/18/shoten9_golangtokyo/)

技術書典10は限界が来て参加できなかったが、今年も2回寄稿できた。  
育児が有り業務外の同期的なイベントに参加するのが困難になってきたので、ブログや技術同人誌のような非同期的な活動に力をいれようかなと思っている。
来年はサークルデビューして1人で何か書いてみようかな？

## OSS活動
作ったものは先週の記事に書いたとおり。

- [2020年振り返り(GitHub編)#OSS](/2020/12/15/retrospective-on-github-2020/#oss)

OSSに対して全然PRを出すことはなかった。  
来年は引き続きTerraformをいじることが多いので、Terraform ProviderやCI/CDツールにPR出せたらなと思う。

## 個人AWSでECSの運用を始めた
機能的にはまだ何も作りこんでいないが、個人のAWS環境でECSを運用し始めた。  
ひとまずTerraformネイティブにいろいろ作っている。業務では触らない範囲も自分でいろいろやるので学びがいろいろある。

- [TerraformでAWS上にHTTPS化したサブドメインを定義する](/2020/11/07/define_https_subdomain_by_terraform/)
## その他のプライベート
- リモートワーク
    - 3月からほぼリモートワークで働いていた。育児的にリモートワークじゃないと破綻する。来年はどうなるんだろう？
- ボルダリング
    - 昨年は週1.5回の勢いでやっていたのだが1人で外出すること（余裕）がなくなったので辞めてしまった。いつか再開したい。
- 酒
    - 夜の子守があったりするのでやめた。ここ4か月くらいだと3回くらいしか飲んでいない。お酒はやめられないと思っていたが意外となんとかなっている。
- 英語
    - Oxford Bookwormsから改めて始めてみた。変に頑張らない分ちゃんと読書できていて楽しい。
    - English Grammar in useも30unitくらいやったのだが習慣化に失敗してしまったので来年は再開したい。
- 読書
    - 可処分時間（子守のスキマ時間）にできることが読書くらいになったので圧倒的に本を読む量が増えた。
    - これは今年身についた良い習慣なので来年もどんどん読書していきたい
    - とは言え買った本の半分も読めていない（72/147）のでもっと読書していきたい
- Kindle Paperwhite
    - ずっとiPhoneやiPadのKindleアプリで読書していたが、Kindle Paperwhiteを買ったら本当に読書へ集中できるようになった
    - [ハンドストラップ](https://amzn.to/3pfWkKu)つけると片手で読める。これがないと読めない

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07HCSQ48P&linkId=6039e317072657f5b5eff83ed3d83378"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B01G8KFDCA&linkId=00e8b4dd78ca29982ec71b03fa71f8e7"></iframe>

# 今年触った技術
## Go
今年もGo中心で頑張っている。来年も軸にしていきたい。  
ただ最近伸び悩みを感じている。書けば書くほど、調べれば調べるほどまだまだだな…と感じるのでもっと突き詰めたい。

- [Go関連記事](/categories/go/)

## Terraform
かなり業務で書くことが多かった。module設計をしたりimportやstate mvしたりそれなりにがっつり書けている。  
terraform管理外の既存のdev/stg/prd環境やネットワーク構成といかに共存するか？というようなことは業務でしか味わえないので楽しい。

## AWS
ECS周辺くらいしか触っていなかったが、今年はRoute53やELB、Kinesisまで設定することが増えてきたので範囲が広がってきた。  
それなりに動かしているサービスも増えてきて、来年は共通化や基盤づくりを意識してインフラ構築に取り組んでいきたい。

## PHP
春先のことになるが登壇もすることができた。  
冬になってから触ることが少なくなってしまったが、来年は再びがっつりやる気がするので手になじませたい。

## Python
こちらも同様最近触れていないが来年は再びがっつりやる。

# 2019年のTryの振り返り
- [2019年振り返り](/2019/12/22/retrospective-2019/)

2019年の振り返りとその結果は以下の通り。

|評価|内容|詳細|
|---|---|---|
|○|3月までにGAEで家族用のLINE Botを作る| 軽く動くものは作れた
|△|英語の本を2冊完読する|Oxford Bookworms stage 1から頑張っている|
|△|PHPやPythonのアウトプットをする|ブログもそこそこ書けた|
|-|家庭とエンジニア生活の両立|前述の通り|
|✗|週1.5回のボルダリング、会社のトレーニングマシーンで定期的な筋トレ|諦めた|
|✗|Vue.jsにも挑戦する|フロント触る機会がなかったので低優先のまま未着手|
|✗|ブログの月間PVが3万超える|今は1万PVいくかいかないか…|

正直目標の内容を忘れていたというどうしようもない感じである。  
毎月の目標が年間目標に紐付いているか確認しないまま目標設定をしていた。  
作る系のアウトプットが出ていないので2020年は定期的に進捗確認をしようと思う（今年はほとんど言うだけになっていた）。

# 2020年のまとめ（KPT）

## Keep
- PHPのカンファレンスでプロポーザルを通して登壇できた
- 技術書典に2回登壇した
- Goのブログ記事で3桁はてブ取得できた
- OSSを3つ作った
- 自分のOKRを諦めてチームの必達OKRにコミットしたりできた
- 新規プロダクトのTerraform立ち上げができた
- 個人AWSでECSの運用を始めた
- 読書量が圧倒的に増えた

## Problem
- OSSにPRが全然だせなかった
- ブログのPVが下がり続けている
- 積ん読が多い
- 春以降今までの貯金でやり過ごしている感が出てきた
    - 技術的にあまり成長していない感がある
- データベースの話になると無力になる

## Try
- 35歳エンジニア引退説への闘争（育児とキャリアの両立）
    - 来年35歳なので（子供ができて、こうして意図せず引退していくのかと痛感している）
    - 可処分時間での学習を諦めない
- データベースの学習
    - 後回しにしていたDB本を周回する
- ひとつ突き詰めて実装してみる
    - 動いて終わりではない何かを作ってみる
    - なんらんかのパーサーかな？
- 毎月5冊本を読む
    - booklogで管理
- 個人AWS環境のECSでNew Relicを試す
    - ふわっと触って理解した気分になっている技術が多いので、来年はo11yを突き詰める
- ISUCON予選突破
    - 年単位で計画的に過去問などを解いていく
- 英語と向き合う
    - 今年こそA Philosophy of Software Designを読み切る
- 毎月の目標をこのTryから落とし込んで2021年で達成したことを忘れない
    - 途中で目標が変わることは問題ない

# 2020年の記録

以下、勉強会や読書の記録。

## 参加した技術イベント
- [Observability Japan Online #1](https://observability.connpass.com/event/168837/)
    - [Observability Japan Online #1 に参加してo11y成熟モデルを学んだ #o11yjp](/2020/03/21/observability_japan_online/)
- [(Online only) Go 1.15 Release Party in Japan](https://gocon.connpass.com/event/186317/)
- [Go Conference '20 in Autumn SENDAI](https://sendai.gocon.jp/)

その他オンラインイベントの録画を後追いで見たりはしている。

## 登壇したイベント
- [PHPerKaigi2020](https://phperkaigi.jp/2020/)
    - [[発表資料] PHPerKaigi2020 PHPでつくるインタプリタ入門 #phperkaigi](/2020/02/10/phperkaigi2020/)
    - [PHPerKaigi2020に4名のメンバーが登壇・プラチナスポンサーとして協賛しました](https://devblog.thebase.in/entry/2020/02/17/155223)
- [【BASE社合同勉強会】コネヒトマルシェオンライン「事業を支えるWeb開発](https://connehito.connpass.com/event/176890/)
    - [[発表資料]ゆるふわ分散トレースはじめました #コネヒトマルシェ](/2020/06/04/connehito_marche/)

登壇回数は2回、登壇時間の合計は40分だった。

## 2020年に読んだ書籍
- https://booklog.jp/users/budougumi/stats

無難だが今年読んでよかった本は以下。
「データ指向アプリケーションデザイン」はかなり難易度が高かったので何度も読み直して咀嚼したい。  
今さら読んだ「アジャイルソフトウェア開発の奥義」は最高だった。エンジニア2,3年目くらいに読み込めばかなり強くなれそう。  
エンジニアリング本以外だと、「嫌われる勇気」と「インテグラルシンキング」はかなり自分の考え方が変わったのでよかった（これももっと前に読みたかった感）。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873118700&linkId=bf8091e7accb6f7b04ab68685c988c09"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B00UX9VJGW&linkId=05ce94d8486d80d4cd24c1e4d0b02958"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B086JJNDKS&linkId=4c375a3516abbca7f413123906ac1fa2"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4797347783&linkId=570e0560904d3688788740ea49f7e96f"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B00IHR7ZVK&linkId=5b4b14fed5e1bec8cd387e111defb410"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B00H7RACY8&linkId=7dcf02f8dcd1ae705225cb150202cf0f"></iframe>

- [「わかる」とはどういうことか　――認識の脳科学](https://amzn.to/3p7uL5X)
  - [[書評] 「わかるとはどういうことか」を読んで”わかる”がどんなことかわかった](/2020/09/17/review_wakarutoha/)
- [Engineers in VOYAGE ― 事業をエンジニアリングする技術者たち](https://amzn.to/2KHCuch)
  - [[書評] Engineers in VOYAGEを読んで事業に立ち向かう技術者を知る #voyagebook](/2020/08/16/review_voyagebook/)
- [インテグラル・シンキング　――統合的思考のためのフレームワーク](https://amzn.to/3apfd9P)
  - [[書評] 情報と向きあい、より深い思考をするためにインテグラル・シンキングを読んだ](/2020/05/14/review-integral-thinking/)
- [ghq handbook](https://leanpub.com/ghq-handbook)
  - [[書評] ghq handbookを読んでghqコマンドに再入門する](/2020/01/11/review-ghq-handbook/)

簡単な感想はブクログで書くことにした。

- [やり抜く人の９つの習慣　コロンビア大学の成功の科学](https://amzn.to/3mB85cX)
    - https://booklog.jp/users/budougumi/archives/1/B0732RRCWX
- [やる気が上がる8つのスイッチ](https://amzn.to/2WuGMWO)
    - https://booklog.jp/users/budougumi/archives/1/B07ZCZYW2Y
- [ザ・ゴール　コミック版](https://amzn.to/2KcXZBW)
- [データ指向アプリケーションデザイン ―信頼性、拡張性、保守性の高い分散システム設計の原理](https://amzn.to/2J4BCxK)
- [分散システムデザインパターン ―コンテナを使ったスケーラブルなサービスの設計](https://amzn.to/2KEcluX)
- [マイクロサービスパターン［実践的システムデザインのためのコード解説］](https://amzn.to/37Bl0XU)
- [ドメイン駆動設計入門 ボトムアップでわかる！ドメイン駆動設計の基本](https://amzn.to/2WxNTxO)
    - https://booklog.jp/users/budougumi/archives/1/B082WXZVPC
- [嫌われる勇気](https://amzn.to/37BRphc)
- [理論から学ぶデータベース実践入門 ~リレーショナルモデルによる効率的なSQL](https://amzn.to/3h0Hncs)
    - https://booklog.jp/users/budougumi/archives/1/4774171972
- [実践版ＧＲＩＴ　やり抜く力を手に入れる](https://amzn.to/2Kmoepp)
    - https://booklog.jp/users/budougumi/archives/1/B078X9K168
- [React.js & Next.js超入門 ](https://amzn.to/37EKRhT)
- [いつも「時間がない」あなたに　欠乏の行動経済学](https://amzn.to/3nALSwP)
    - https://booklog.jp/users/budougumi/archives/1/B012LXVDM2
- [漫画 バビロン大富豪の教え　「お金」と「幸せ」を生み出す五つの黄金法則](https://amzn.to/2WtaoUK)
- [テスト駆動開発](https://amzn.to/37wogns)
- [２１　Ｌｅｓｓｏｎｓ　２１世紀の人類のための２１の思考](https://amzn.to/2KKSIRC)
    - https://booklog.jp/users/budougumi/archives/1/B07Z8ZY42X
- [マンガでよくわかる エッセンシャル思考](https://amzn.to/2WJB97D)
    - https://booklog.jp/users/budougumi/archives/1/B06XP9632T
- [初めてのGraphQL ―Webサービスを作って学ぶ新世代API ](https://amzn.to/37xObuQ)
- [IT業界を楽しく生き抜くための「つまみぐい勉強法」](https://amzn.to/3nCFsgy)
    - https://booklog.jp/users/budougumi/archives/1/477414259X
- [その仕事、全部やめてみよう――１％の本質をつかむ「シンプルな考え方」](https://amzn.to/2WzcF0u)
    - https://booklog.jp/users/budougumi/archives/1/B08CQRKHKR
- [英語は「英語で学ぶ」とうまくいく 英語で考える力をつける14段階式英語学習法](https://amzn.to/3p9zAMh)
    - https://booklog.jp/users/budougumi/archives/1/B07KG71127
- [失敗の本質](https://amzn.to/3nAgYEF)
- [アジャイルソフトウェア開発の奥義 第2版 オブジェクト指向開発の神髄と匠の技](https://amzn.to/2WxOlMw)
    - https://booklog.jp/users/budougumi/archives/1/4797347783
- [ユースケース駆動開発実践ガイド](https://amzn.to/2WyGj61)
- [アクション リーディング　1日30分でも自分を変える“行動読書“](https://amzn.to/3h1T82n)
    - https://booklog.jp/users/budougumi/archives/1/B01G6U99B8
- [実践ドメイン駆動設計](https://amzn.to/2WtaNGK)
- [サーチ・インサイド・ユアセルフ ― 仕事と人生を飛躍させるグーグルのマインドフルネス実践法](https://amzn.to/3asaRyB)
- [カード決済業務のすべて―ペイメントサービスの仕組みとルール ](https://amzn.to/2Kcaz4l)
- [決済サービスとキャッシュレス社会の本質](https://amzn.to/3rgkvdN)
- [今、ここを生きる ──新世代のチベット僧が説くマインドフルネスへの道](https://amzn.to/3rhcyEX)
    - https://booklog.jp/users/budougumi/archives/1/B01M0QRJ3S

## 2020年に読み終わらなかった書籍
- https://booklog.jp/users/budougumi/all?category_id=all&status=4&rank=all&sort=sort_desc&display=front

毎年ちゃんとリストにしていたが、完全に破綻してるので今年から止める。

# 関連
- [2020年振り返り(GitHub編)](/2020/12/15/retrospective-on-github-2020/)
- [2019年振り返り](/2019/12/22/retrospective-2019/)
- [2019年振り返り(GitHub編)](/2019/12/14/retrospective-on-github-2019/)
- [2018年振り返り](/2018/12/27/retrospective-2018/)
- [2018年振り返り(GitHub編)](/2018/12/15/retrospective-on-github/)
- [2017年振り返り](/2017/12/30/retrospective-2017/)
- [2017年振り返り(GitHub編)](/2017/12/27/retrospective-on-github/)
