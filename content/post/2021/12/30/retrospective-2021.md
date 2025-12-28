+++
title= "2021年振り返り"
date= 2021-12-30T09:30:52+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["esssay"]
tags = ["Retrospective"]
keywords = ["振り返り"]
twitterImage = "twittercard.png"
+++

2021年を振り返ってみる。GitHubベースの振り返りは別記事にまとめた。

- [2021年振り返り(GitHub編)](/2021/12/26/retrospective-on-github-2021/)

<!--more-->

# 主な出来事

## ブログなどのKPI
毎週Twitterのフォロワー数とブログのスコアを集計している[^kpi_issue]。

[^kpi_issue]: twitterのフォロワー数はSocialDogで取るようになった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/blog-kpi-collector" data-iframely-url="//cdn.iframe.ly/13K41P4?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

今年は以下のような数値だった。

|集計日|Twitter フォロワー数|ブログの週間PV|ブログの累計はてブ数|
|---|---|---|---|
|2020/12/27|1067|1384|2739
|2021/03/28|1092|3643|2794
|2021/06/27|1107|2801|2805
|2020/09/27|1155|2384|2883
|2020/12/26|1204|2577|2910

Twitterのフォロワーが1,200人を超えた。  
執筆していることを公表したあと結構フォローしてもらった。期待に答えられるよう頑張りたい。  
最近あまり技術的なブログ記事を書けていないのでtweetだけでもいい情報届けられないか少し意識している。こういうのとか。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">最近Go Playgroundでdev branch使えるようになったのでブラウザでもリリース前のgenericsとか簡単に試せるようになりました。<a href="https://t.co/okQwOEFASJ">https://t.co/okQwOEFASJ</a> <a href="https://t.co/dhpBzBgaqC">pic.twitter.com/dhpBzBgaqC</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1473844460794839042?ref_src=twsrc%5Etfw">December 23, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


2021年のブログは38記事執筆した。  
年間PV数は2021/12/26現在で140,100PVくらいだった。2020年の年間PVと比較すると1万PVくらい下がっている。
はてブされたりニュースメディアに載るような記事が書けなかった。はてブが多かった記事は以下。

- [Goでプログラムを終了するときのお作法 - My External Storage](/2021/06/30/which_termination_method_should_choose_on_go/)

また、会社のブログにも3記事寄稿、1記事コメントした。今年の会社アドベントカレンダーの中ではてブ数No.1になったのでそれなりにバリューは出せた。
<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fisucon11" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>
<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fnrug_vol0" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>
<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fdevops_key_metrics_on_newrelic" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>
<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fauto_generated_er_graph_by_tbls_and_github_actions" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

## New Relic User Groupで登壇した
19時以降のイベントは家庭の事情で参加できないので登壇も諦めていたが、こちらのイベントは夕方開催だったので登壇できた。  
自作OSSを紹介しただけだがナマっていたのでちょうど良かったかと思う。久しぶりに登壇資料つくるとやはりむずかしい（久しぶりじゃなくても難しい）。

<div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.1972%;"><iframe src="https://speakerdeck.com/player/0c0b3145cd014493b38e063d680b7b84" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no" allow="encrypted-media;"></iframe></div>

## OSS活動
作ったものは次の記事に書いたとおり。業務に耐えうるいろいろを作れたのはよかった。

- [2021年振り返り(GitHub編)](/2021/12/26/retrospective-on-github-2021/#oss)

## ISUCON11に参加した
[ISUCON11予選に参加した（最終スコア47,995点 79位） #isucon](/2021/08/24/isucon11q/)

今年はついに参加した！思ったより点数が取れたけれど終盤は時間切れの前に手札がなくなってしまった。  
ここ数年参加する参加すると言って全然参加しておらず、今回も育児を理由に諦めるところだった。ちゃんと参加できてよかった。  
来年こそは本戦に出場したい。

## 育児はたいへん
去年産まれた双子は12月で1歳半になる。まだ保育園に行っていないのでずっと自宅で育児をしている。
子供優先なので今年は仕事以外に一人で外出したのは映画館に3回行ったのと床屋くらいだった。  
私は毎月の残業が10時間未満なのだが就業時間より家事や育児の拘束時間が長く、最近まで夜もずっと夜泣きしていたので眠れない日々が続いた。  
（うちの子たちだけかもしれないが）双子は公園にいくのもハード（ワンオペでは不可能）なのでいろいろと辛い。

![育児は大変](/2021/12/29_sleep.jpg)


## 引っ越しした（家を買った）
- 去年双子が産まれた
- リモートワークが続いている
- 昭和生まれ（今年35歳）なのでそろそろローン組んでおきたい

上記の状況で1LDKに住み続ける（WFHする）のは辛かったし、そろそろローンを組まないと老後が大変そうなので家を買った。
去年育児休暇を4か月していて額面年収が下がっていたのでローン審査はそれなりに大変だった。

定期的に話題になるソフトウェアエンジニアの自宅購入記事は注文住宅がほとんどだけれども、私の場合は分譲住宅を買った。
同じ条件（駅徒歩圏内駐車場付き4LDK（結果的には5LDKになった））をマンションで探すと億超えるし育児に追われながら満足行く家の設計などできるわけなく、分譲住宅以外は諦めた。
分譲住宅は分譲住宅で次のようなメリットがあった[^2]。

[^2]: ただn=1の結果論なので全部の分譲住宅がそうとは言わない。

- 複数住宅がまとめて設計されるので複数の分譲住宅で空間設計がされる
    - 隣家と密着してるとか日当たりが悪いとかない
- 人生設計やフェーズが似ている人たちが集まる
    - 小学生低学年未満の子供のいらっしゃる家庭が多くてよかった。

引っ越す前はAmazonのダンボールでずっと働いていた。
![引越し前](/2021/12/before_desk.jpeg)

引っ越してからは書斎ができたのでFLEXISPOTとコンテッサを揃えた。圧倒的に快適になった。道具は大事。
![引越し後](/2021/12/after_desk.jpg)

ちなみにFLEXISPOTはAmazonのサイバーマンデーなどで3~5割引になるので、購入を検討している人はセールを狙ったほうが良い。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B092VX5RK7&linkId=81a9e85c9bdf5bae68f646c75cfa2ea6"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B071P6VTZL&linkId=576676cbb79765fcf3f95ed707889b2e"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B08XNFN1WM&linkId=6ff942e96139d8003170921b53a5d3e2"></iframe>

## 執筆を依頼された
ありがたいことに[C&R研究所](https://www.c-r.com/)様から執筆のお話をいただいたので執筆している。  
本当は年内に初稿を提出したかったのだが公私に渡っていろいろあったのでほとんど書けなかった。来年の1Qはなんとしても初稿を書き終わる。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">実は来年発売に向けて書籍執筆中です！！（発売日は調整中） <a href="https://t.co/r9nsPU5Ge8">https://t.co/r9nsPU5Ge8</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1470011071633440768?ref_src=twsrc%5Etfw">December 12, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## その他のプライベート
- リモートワーク
    - 相変わらずリモートワークだった。最近は週イチで出社していてだいぶ気分転換になる。
- 酒
    - 今年はビール2杯分くらいしか飲まなかった。ノンアルコールビールも惰性で飲んでただけだったのでほぼやめた。
- 英語
    - [[書評] A Philosophy of Software Design を読んだ。複雑性を理解し、戦う術を身につけた](/2021/12/17/review_aposd/)
    - 英語の技術書を一冊読み切ったのはだいぶ自信につながった
- 読書
    - Audibleをわざわざ買わなくてもKindleで読み上げればよい。ながら読みがはかどった。
      - [Amazon AlexaアプリでKindleを読み上げて積ん読を消化する（ながら読書をする）](/2021/03/08/read_kindle_by_alexa/)

# 今年触った技術
## Go
- [Go関連記事](/categories/go/)

シミュレーションテストを設計・実装したりOSSを作ったりはしていた。  
OSS活動で普段書かない静的解析をするのは楽しかった。ASTを予想してコードが「読めた」ときは楽しい。
Go1.18についていけるかが最近の不安。

仕事ではBASEカードという決済サービスをGoで実装してリリースした（シミュレーションテストもこれに関連して作った）。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://cp.thebase.in/basecard" data-iframely-url="//cdn.iframe.ly/berN2KD?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

実装面では以下のようなハードルがありなかなかタフな設計だった。  

- いくつかの外部の提携先と実現しており、会社をまたいだ論理トランザクションがある
- 社内でも複数DBが関わっていて分散トランザクションがある
- 応答速度が求められる同期処理
- 結果整合性が求められる非同期処理

サービス間の認識齟齬が発生したり、命名付ひとつでかなり認知負荷があがるのを実感したりというアルゴリズムや構造ではないチーム開発みたいなところでも難しさを実感する1年だった。  
ゼロからNew Relic Nativeな実装をしたのもはじめての年だった。今はもう分散トレース、ログがないと何もわからんとなっている。もっと使いこなしたい。

## AWS
あまり新しいサービスは触らなかった。  
SNS/SQS/Lambdaを使った非同期設計をやった。それなりに考えることがあり難しい。  
御多分に漏れずリトライ処理で非冪等なAPIを複数回叩いてエラーを発生させたりした。

## PHP&Vue
あまり触れなかったし、レビューも他人任せみたいな状態だったので変えていかないといけない。
チーム構成や走るプロジェクトの性質が変わるので来年はがっつり触る予定。


# 2020年のTryの振り返り
- [2020年振り返り](/2020/12/27/retrospective-2020/)

今年のKPTを考える前に去年のTryにちゃんと取り組めたか確認しておく。

|評価|内容|詳細|
|---|---|---|
|△|35歳エンジニア引退説への闘争（育児とキャリアの両立）| 仕事は評価してもらっているが、両立できてるかというとどうだろう？
|✗|データベースの学習|詳説データベースなど積読で終わった。ISUCON的な意味では知識がついた
|○|ひとつ突き詰めて実装してみる|[静的解析はそれなりにできた](https://github.com/budougumi0617/nrseg)
|✗|毎月5冊本を読む|平均3冊くらいしか読んでいない|
|✗|ISUCON予選突破|[予選敗退したが初参加にしてはそこそこ点数取れた気がした](/2021/08/24/isucon11q/)|
|○|英語と向き合う|APoSD読了
|○|毎月の目標をこのTryから落とし込んで2021年で達成したことを忘れない|月間目標立てるときちゃんとすり合わせした|

子供が1歳すぎたらプライベートの時間が復活すると思っていたがそんなことはなかった。双子育児を舐めていた。

# 2021年のまとめ（KPT）

## Keep
- 業務で使えるOSSを複数作れた
- 業務でデカいリリースに貢献できた
    - https://cp.thebase.in/basecard
- ISUCONでそれなりの成果を出すことができた
- 仕事しながら家の購入などを進めることができた
- TiDBにいくつかPRを出すことができた
    - 下期はほとんどOSS活動できていなかったので少し気晴らしになった
- Notionでタスク管理を始めた
- Streaksで習慣づけを始めた
    - https://apps.apple.com/jp/app/streaks/id963034692

ずっとMarkdownでメモやタスク管理をしていたがさすがに限界を感じてNotionを使い始めた。  
リマインダなどはできないが、使うツールを増やしたくないのでこれでよかったかなと思う。  
自分は習慣づけができると思っていたが結構意思が弱いこともあったのでブログ仲間に教えてもらったStreaksを使うようになった。今のところ結構有用で「やらない」もうまく継続できている。

## Problem
- 下期はプロセス改善などの中長期タスクに取り組めなかった
- 実装が遅いせいでリスケが必要になることが何回かあった
- データベースの学習など長期的な学習計画を履行できなかった
- 読書があまりはかどらなかった
    - 下期は技術書をほとんど読んでいなかった
- 執筆活動の進捗が著しく悪かった

## Try
- 公私両立
    - 保育園が始まるとまた生活習慣が変わってくるので。
    - いろいろ事情があり送迎は誰がやるとかまだ決まっていないのでふわっと。
- 出版!!!!!!
    - 締切を守る
- フロントエンドの技術習得
    - だいぶ他人任せが多かったが、来年は手をちゃんとうごかす
- チームの文化をつくる
    - プロセス改善と文化醸成に力をいれる割合を増やす
- 毎月5冊本を読む
    - 継続目標
- 英語の技術書を2か月に1冊読む
    - オライリーのサブスクに申し込む
- ISUCON予選突破
    - 来年こそは
- デイキャンプにいく。

手応えを感じた面もあるが総評的にはやらなければいけないことができていなかった。  
ちょっとぼやっとしている目標が多いが、年末年始でもう少し具体的にしておきたい。  


来年はこどもが2歳になってある程度分別がつくようになるはずなので泊まりはダメでもせめてデイキャンプに行きたい。  
今年のアドベントカレンダーはキャンプのアドベントカレンダーが一番楽しみだった。参考になる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://adventar.org/calendars/6236/embed" data-iframely-url="//cdn.iframe.ly/TUXlEOS"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 2021年の記録

以下、勉強会や読書の記録。

## 参加した技術イベント
- [第36回横浜Go読書会](https://yokohama-go-reading.connpass.com/event/206437/)
    - [[Go] stringの比較でヌルポのpanicが発生する（こともある） #横浜Go読書会](/2021/03/31/go-string-null-pointer-panic/)
- [第37回横浜Go読書会](https://yokohama-go-reading.connpass.com/event/208878/)
- [第38回横浜Go読書会](https://yokohama-go-reading.connpass.com/event/212311/)

夜は基本無理でGoのリリパも参加できなかったのが残念。

## 登壇したイベント
- [New Relic User Group Vol.0](https://newrelic.com/jp/events/2021-09-15/nrug1)
    - [New Relic User Group Vol.0で登壇しました #NRUG](/https://devblog.thebase.in/entry/nrug_vol0)

来年はもっと登壇したい。

## 2021年に読んだ書籍
- https://booklog.jp/users/budougumi/stats

圧倒的にA Philosophy of Software Designが良かった。俗に言う「臭い」を定性的に説明できるようになったと思う。  
失敗の科学、1兆ドルコーチ、WHO YOU AREは「困難に立ち向かう気持ち」「困難と戦う術」が書いてありとてもモチベーションがあがった。  
今年も「なんで今まで読まずに生きてきてしまったんだ…」と思える本ばかりでよかった。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B09B8LFKQL&linkId=20b421e5f82a8f7c2cf1687e71de6687"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07ZCY5BXF&linkId=acdc6a2dabbb2c3c406691a3aad357c0"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B01MU364ID&linkId=4c21e33f876f88552b5c87c9eb116f66"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B086KX8LHZ&linkId=0c3cd045443d8843cbebdb24212947aa"></iframe>

- [どんな仕事も「25分+5分」で結果が出る　ポモドーロ・テクニック入門](https://amzn.to/3EuCGBy)
    - [[書評] ポモドーロ・テクニック入門を読んで正しくポモドーロをしようと反省した](/2021/05/23/review_pomodoro_technique/)
- [具体と抽象](https://amzn.to/32CbYtv)
    - [[書評] 具体と抽象を読んで抽象化思考の整理と向上方法を学んだ。](/2021/06/28/review_concrete_and_abstract/)
- [1兆ドルコーチ――シリコンバレーのレジェンド　ビル・キャンベルの成功の教え](https://amzn.to/3mzTIYX)
    - [[書評] 1兆ドルコーチを仕事や仲間に対する姿勢について感銘をうけた](/2021/06/29/review_million_dollar_coach/)
- [HARD THINGS　答えがない難問と困難にきみはどう立ち向かうか](https://amzn.to/3en8zS8)
    - [[書評] HARD THINGSを読んで他者との協調・組織運営について考えた](/2021/07/27/review_hard_things/)
- [早く正しく決める技術](https://amzn.to/3z36QuH)
    - [[書評] 早く正しく決める技術](/2021/08/31/review_technique_to_decide_quickly/)
- [フリーズする脳　思考が止まる、言葉に詰まる](https://amzn.to/3pruduF)
    - [[書評] フリーズする脳　思考が止まる、言葉に詰まる](/2021/09/24/review_frozen_brain/)
- [失敗の科学　失敗から学習する組織、学習できない組織](https://amzn.to/3emdQcK)
    - [[書評] 失敗の科学 失敗から学習する組織、学習できない組織](/2021/09/30/review_black_box_thinking/)
- [他者と働く──「わかりあえなさ」から始める組織論](https://amzn.to/32jndYl)
    - [[書評] 他者と働く─「わかりあえなさ」から始める組織論](/2021/10/26/review_work_with_others/)
- [恐れのない組織――「心理的安全性」が学習・イノベーション・成長をもたらす](https://amzn.to/30Yuj3E)
    - [[書評] 恐れのない組織 「心理的安全性」が学習・イノベーション・成長をもたらす](/2021/10/31/review_the_fearless_organization/)
- [Who You Are（フーユーアー）君の真の言葉と行動こそが困難を生き抜くチームをつくる](https://amzn.to/32Gobha)
    - [[書評] WHO YOU AREを読んだ。文化とはルールではない。行動だ。](/2021/11/28/review_who_you_are/)
- [A Philosophy of Software Design, 2nd Edition](https://amzn.to/3pruvlf)
    - [[書評] A Philosophy of Software Design を読んだ。複雑性を理解し、戦う術を身につけた](/2021/12/17/review_aposd/)

その他簡単な感想、感想なしの本。

- [フィードバック入門 耳の痛いことを伝えて部下と職場を立て直す技術](https://amzn.to/3en9hyM)
    - https://booklog.jp/item/1/B06VVQ8V36
- [直感力](https://amzn.to/30ZPJNX)
    - https://booklog.jp/users/budougumi/archives/1/B00DMXMWHG
- [まんがで身につく 続ける技術](https://amzn.to/3EtBnTu)
    - https://booklog.jp/users/budougumi/archives/1/B01JADEHTW
- [そろそろNotion](https://amzn.to/3qthoiL)
    - https://booklog.jp/users/budougumi/archives/1/B09JS3D9C6
- [Webサービスチューニングコンテスト ISUCONのススメ](https://amzn.to/3quFv0P)
- [論理的思考のコアスキル](https://amzn.to/3FFCfWD)
- [達人に学ぶDB設計 徹底指南書](https://amzn.to/3EzbBgI)
- [Amazon Web Services エンタープライズ基盤設計の基本](https://amzn.to/3z6yNln)
- [ハイ・コンセプト「新しいこと」を考え出す人の時代―――富を約束する「６つの感性」の磨き方](https://amzn.to/3pwpsAg)
- [１日ひとつだけ、強くなる。](https://amzn.to/3pzgVN5)
- [新 コーチングが人を活かす](https://amzn.to/3EtxHkM)
- [データマネジメントが30分でわかる本](https://amzn.to/3eu8MCW)

# 関連
- [2021年振り返り(GitHub編)](/2021/12/26/retrospective-on-github-2021/)
- [2020年振り返り](/2020/12/27/retrospective-2020/)
- [2020年振り返り(GitHub編)](/2020/12/15/retrospective-on-github-2020/)
- [2019年振り返り](/2019/12/22/retrospective-2019/)
- [2019年振り返り(GitHub編)](/2019/12/14/retrospective-on-github-2019/)
- [2018年振り返り](/2018/12/27/retrospective-2018/)
- [2018年振り返り(GitHub編)](/2018/12/15/retrospective-on-github/)
- [2017年振り返り](/2017/12/30/retrospective-2017/)
- [2017年振り返り(GitHub編)](/2017/12/27/retrospective-on-github/)

