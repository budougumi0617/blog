+++
title= "@kakakakakkuさんのブログメンティーを卒業した（2回目）  #ブログメンタリング"
date= 2019-04-02T13:46:02+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["esssay"]
tags = ["Retrospective", "Kakakakakku"]
keywords = ["ブログメンティー", "ブログメンタリング"]
twitterImage = "twittercard.png"
+++


1月から3月まで[@kakakakakku](https://twitter.com/kakakakakku)さんのブログメンタリングを受けていた。  
私は昨年の4月から6月までも1回メンタリングしてもらっていたので、この記事はブログメンタリング2回目の振り返りとなる。
<!--more-->

# TL;DR
- 毎週書いているのにPVが伸びない = 記事の質がよくない？と思い、2回目のブログメンタリングを受けた
- 見やすいブログにするための改修や、KPIの集計を始めた
- 月間PVは3ヶ月の間、毎月1,3000PV以上をキープできた
- 被はてブ数は3ヶ月で約1,300増えた
- 念願だったGo（技術記事）で100はてブを超えることができた
- 今後はOSS開発や登壇などのアウトプットをメインにおいて、補足的にブログも書け続けられたら良いなと思う（もちろん週一で）
  - アウトプットはいいぞ！！

# なぜ2回目のブログメンタリングを受けたのか
[@kakakakakku](https://twitter.com/kakakakakku)さんは一部（？）でアウトプットおばけとして有名な方だ。無償でブログメンターをされている。

<script async class="speakerdeck-embed" data-id="bf9871ac31334c53a049ca13c971f683" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

  
まず、私は昨年の4月から6月に[@kakakakakku](https://twitter.com/kakakakakku)さんにブログメンターになっていただき、一度ブログメンタリングを受けていた。  
1回目のブログメンタリングを受けたあとの感想や結果報告はこちら。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://budougumi0617.github.io/2018/07/10/graduate-blog-mentoring-by-kakakakakku/" data-iframely-url="//cdn.iframe.ly/JHjDkY7"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

ブログメンタリングを受けたあとも継続して週に1回ブログ記事を書いていたので、完全にブログへの記事投稿を習慣化をすることはできた。（月に一回の登壇などもしていた）。  
が、アクセス数が伸び悩んだりしていて、昨年末は以下のセルフ目標をクリアできず伸び悩んでいた。

- 月間1万PVを達成すること（8,000PV~9,000PV程度で頭打ちしていた）
- ホッテントリするくらいはてブがつく良質な記事を書くこと（最高で30はてブ弱）

「好きなこと書くのがブログ」はもっともだし正しいと思うのだが、自分の中で「PV（はてブ）が伸びない = 読まれていないということは内容があまりよくない記事なのではないか？」という悩みを抱えていた。

そんな中、[@kakakakakku](https://twitter.com/kakakakakku)さんの以下のTweetを見て、もう一度ブログメンティーになることを決めた。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">ブログメンター卒業生がもう1度メンタリング応募をしてくれるパターン，今まで1度もないけど，それはそれで超刺激的にできそう🎅</p>&mdash; カック@ブロガー / k9u (@kakakakakku) <a href="https://twitter.com/kakakakakku/status/1068804025598980096?ref_src=twsrc%5Etfw">2018年12月1日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 2回目のブログメンタリングで何をしたのか
今回のブログメンタリングでは以下のことに取り組んだ。

## 言い渡された課題
1回目のブログメンタリング（他の方のブログメンタリング）と同様に、ブログメンタリング開始時に20個のネタ出しをした。
それとは別に2回目の私の場合は以下の課題も課せられた。

- ブログサイト内のHTTPリンクの撲滅（完全HTTPS化）
- サイドバーメニューの改修
- サイト内検索の追加
- 記事に対するOGP画像の追加
- 記事に対するTwitterカードの追加
- ヒートマップ分析の導入
- 50以上のはてブ、100以上のはてブがつく記事を作ること
- Goの記事でホッテントリ（100はてブ以上）
- KPIとして毎週Twitterフォロワー数、週間PVを集計すること

正直私は「SEO対策なんて気にしない！わしは文章で勝負するんや！」みたいな変なこだわり（逃げ）をしていたのでSEO対策を何も真面目にやっていなかった。
1月はまず自分のブログの改修を行なった。

## 課題に対してやったこと
私のこのブログは[Hugo](https://gohugo.io/)というOSSで生成しているため、以下の対策を行なった。

- [HTTPSで公開しているHugoで一部のページがHTTPでページ遷移してしまう #hugo](/2019/01/06/redirect-by-http-on-hugo/)
- [HugoでOGP(Facebook用のアイキャッチ画像)を設定する #hugo](/2019/01/05/set-ogp-in-hugo-blog/)
- [Hugoのブログ記事にTwitter Card(アイキャッチ画像)を設定する #hugo](/2019/01/04/set-twitter-image-in-hugo-blog/)
- [HugoブログにUser Heatを設置してヒートマップ分析をする](/2019/01/21/use-heatmap-by-userheat/)

導入・利用するようになったサービスは以下。

- iframely
  - https://iframely.com/
  - （リンク先に依存するのだが、）リッチな埋め込みリンクを提供してくれる
- Card validator
  - https://cards-dev.twitter.com/validator
  - Twitterカードがどのように見えるのか判定してくれる
- シェアデバッガー(要facebook開発者登録)
  - https://developers.facebook.com/tools/debug/sharing/
  - OGP画像が適切に設定されているか、どう見えるのか判定してくれる
- User Heat
  - https://userheat.com/
  - サイトのPV数が月間30万PVまでならば無料でヒートマップ分析をしてくれる。

また、毎週自動集計をするためにGoogle App Script(GAS)を書いた。

- [GoogleスプレットシートでWebサイトのKPIを集計するためのOSSを作った #blog #gas](/2019/01/26/publish-blog-kpi-collector/)

GASを書くのは初めてだったので試行錯誤だったが、OSS化して公開している（starをいっぱいもらえていて嬉しい）。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/blog-kpi-collector" data-iframely-url="//cdn.iframe.ly/VvOUsAg"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


あとは自分の興味のある技術分野で新発表がされたときは、発表数日以内に投稿したりした。

- [GitHub Actionsで特定のブランチのときのみワークフローを実行する #github #actions #ci](/2019/01/11/filter-branch-in-github-actions/)
- [BetaリリースされたGoのGoogle Cloud Functionsを試す #gcp #golangjp](/2019/01/17/try-go-on-google-cloud-functions/)


# メンタリングを受けた結果どう変わったのか
メンタリングを受ける前の悩み（目標）は次の2つだった。

- 月間1万PVを達成すること（8,000PV~9,000PV程度で頭打ちしていた）
- ホッテントリするくらいはてブがつく良質な記事を書くこと（最高で30はてブ弱）

ブログの月間PVは以下のようになった。3月はハズれ値（おばけ記事がひとつあった）だが、毎月13,000PV以上達成できた。

|月|月間PV|
|---|---
1月| 14,706
2月|13,220
3月|34,769

![アクセス数の遷移](/2019/04/02_pv.png)

前述したGASスクリプトでTwitterのフォロワー数やPV、ブログの被はてブ数を週次でカウントしていた。集計結果は以下の通り。昨年までは200程度だった被はてブ数は3ヶ月間で1,300増加した。

|集計日|Twitter フォロワー数|週間PV|週間直帰率|累計はてブ数
|---|---|---|---|---|
2019/01/06|261|1175|84.67|220
2019/01/13|272|3310|85.64|273
2019/01/20|280|5571|90.06|476
2019/01/27|284|3048|87.39|495
2019/02/03|306|2741|87.53|504
2019/02/10|311|2971|88.66|510
2019/02/17|327|3911|88.31|622
2019/02/24|336|3181|87.42|640
2019/03/03|340|2992|87.05|648
2019/03/10|354|23624|87.34|1490
2019/03/17|359|3419|87.18|1493
2019/03/24|362|3159|86.68|1500
2019/03/31|373|3583|87.83|1510

はてブ上位の記事は以下の通り。ブログのメインカテゴリであるGoの記事でホッテントリーに載ることもできた（はてブ数は2019/04/02時点のもの）。

- [[書評] アウトプット大全 を一ヶ月試してみて毎日のアウトプット力が着実に向上し始めた](http://b.hatena.ne.jp/entry/s/budougumi0617.github.io/2019/03/03/review-output-encyclopedia/)
  - 833はてブ
- [Google Apps ScriptをTypeScriptで実装する(clasp/TSLint/Prettier) #gas #typescript](http://b.hatena.ne.jp/entry/s/budougumi0617.github.io/2019/01/16/develop-google-apps-script-by-typescript/)
  - 133はてブ
- [Go Modulesの概要とGo1.12に含まれるModulesに関する変更 #golangjp #go112party](http://b.hatena.ne.jp/entry/s/budougumi0617.github.io/2019/02/15/go-modules-on-go112/)
  - 112はてブ
  - [2019年2月16日の人気エントリー](http://b.hatena.ne.jp/hotentry/it/20190216)

PVとはてブという2つの目標に対して、両方目標達成できたので、ブログメンタリング1回目以上に大きな成果を得ることができた。  
PVが増えることで以下の効果を得ることができた。

- Twitterやはてブでフィードバックをもらえるので技術的な間違い・改善点について知ることができる
- ネガティブコメントを咀嚼するメンタル
- 単純にPV増加に比例するモチベーションの増加

また、2月には[人生2回目の20分枠での社外発表を行なった](https://gocon.connpass.com/event/118022/)が、参加者のみなさんから多数指摘やコメントをいただくことができた。（媒体は違うとはいえ、）普段からブログにアウトプットしてストーリーを考えたり、要約を繰り返していたからこそちゃんと資料が作れたかなと思う。

<script async class="speakerdeck-embed" data-id="025e3f94d6ef4685bd8b3cca762b8e38" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# 今後の課題
当初の目的は達成できたが、メンタリング終了時に以下の課題を[@kakakakakku](https://twitter.com/kakakakakku)さんに指摘された。

- 普段の業務での貢献がブログにあまり出ていないこと
- 登壇実績などのコミュニティでの知名度とOSS実績の差
- 登壇など表に出る機会が多いわりにTwitterフォロワー数が少ないこと

どれも痛い指摘ばかりだった。たしかに普段の業務で業務中に取り扱っている話題や事例的な内容が私のブログには少ない。うまく汎化できていなかったり、そもそも[社内のめちゃ強エンジニアのアウトプットにタダ乗り](https://developers.freee.co.jp/entry/service-infra-and-wire)して業務をしているというところもある。もっと単純な技術力や実践導入力を上げていきたい。  
事例などは登壇や社の開発者ブログで会社の名前を背負って話すほうが突っ込んだ内容を話せる（書ける）ので、その技術補足や詳細をうまく自分のブログに落とし込んでいく、などのアプローチになるかなと思う。

また、OSS活動などもその通りで、自分はもともと「あまり技術力がないので運営などでコミュニティにコミットしていこう」という感じだった。  
とはいえそれでもドキュメントに対するコミットなどは出来るはずで、あまり言い訳はできない。そもそもOSSを作ろうという動機となる「なければ作る」が「なければ諦める」になっている部分もあるので、自分や周りのNeedsを深掘りして自分にいま必要なOSSを作っていきたい。  
直近は最近興味がある[analysisパッケージ](https://godoc.org/golang.org/x/tools/go/analysis)を使ったOSS開発などに取り組んでいきたいなと思う。

# 終わりに
ブログメンタリングを受けた背景と実施内容、結果をまとめた。  
今後はOSS開発や登壇などのアウトプットをメインにおいて、補足的にブログを書けたら良いなと思う（もちろん週一で）。  
ブログを書いたりアウトプットをしていると思考の整理になったり多方面からフィードバックを得られる。アウトプットはいいぞ！

# 参考
- [楽しく！アウトプットを習慣化しよう](https://kakakakakku.hatenablog.com/entry/2018/07/06/184236)
- [アウトプット駆動学習を習慣化する](https://kakakakakku.hatenablog.com/entry/2017/12/22/173455)




