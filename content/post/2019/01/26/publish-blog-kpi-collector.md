+++
title= "GoogleスプレットシートでWebサイトのKPIを集計するためのOSSを作った #blog #gas"
date= 2019-01-26T08:01:17+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["blog", "gas"]
tags = ["clasp", "gas", "blog", "kpi", "seo"]
keywords = ["SEO", "ブログ", "GAS", "Google Apps Script", "KPI"]
twitterImage = "2019/01/26-kpi-collector.png"
+++

Webサイト（ブログ）のKPIを集計するGoogle Apps Script(GAS)をシュッと作成するスターターキットのOSSを作った。
一通りの設定をすると、Googleスプレットシートに各情報が書き込まれる。GASのトリガー機能で定期実行すれば、定期的にKPIを自動集計できる。

![集計イメージ](/2019/01/26-results.png)

2019/01/26現在対応している項目は以下。

- Google Analyticsの週間PV数
- Google Analyticsの週間直帰率
- はてなブックマークの総数
- はてなブログの読者数
- Twitterのフォロワー数

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/blog-kpi-collector" data-iframely-url="//cdn.iframe.ly/api/iframe?url=https%3A%2F%2Fgithub.com%2Fbudougumi0617%2Fblog-kpi-collector%2F&amp;key=bc7b75ad5adf2dc500d4b0dee9f92949"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- `Node.js`の実行環境、Googeアカウントがあれば利用できる
- ローカルから`clasp`コマンドを使って簡単にGASとスプレットシートの準備が出来る
- ブログのURLなどをスクリプトのプロパティに設定し、トリガーなどをWebブラウザで設定するだけ
- 全部無料で使えるのでよかったら使ってみてください

# なぜ作ったのか？（誰が使うのか？）
このブログを毎週更新している。去年まではなんとなくGoogle Analyticsのグラフを見ていただけだった。今年はもう少しブログの質をちゃんとみよう、と思いもろもろの情報をGoogleスプレットシートに集計することにした（ちゃんと見ましょう、という指摘を受けたほうが大きいが）。このOSS作成以前に自分で使うためのGASコードは出来てたのだが、他のブロガーの方も同様の集計をしたい・しているということなので一般化して公開することにした。

# 仕組みの概要
このOSSは以下の流れで利用できる。`Node.js`の準備などで詰まらなければ、だいたい30分もかからず利用開始できるはずだ。

- ローカルで`clasp`コマンド（後述）を利用する準備をする
- OSSをダウンロード（`git clone`）する
- `clasp`コマンドを使って、Google DriveにGASのプロジェクトとGoogleスプレットシートを自動生成する
- Webブラウザ上でブログURLなどをスクリプトのプロパティ（GASの環境変数みたいなKeyValue）に設定する
- 実行して権限設定をして、定期実行をするためのトリガーを設定する

使い方の詳細については、リポジトリのREADMEに網羅しているつもりなのでそちらを参照のこと（不足があれば[issueで指摘](https://github.com/budougumi0617/blog-kpi-collector/issues/new)していただければ）。

- https://github.com/budougumi0617/blog-kpi-collector/blob/master/README.md

`clasp`コマンドとは、ローカルでGAS開発をするためのGoogle公式ツールだ。`npm`ライブラリなので、実行には`Node.js`の準備が必要だ。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/google/clasp" data-iframely-url="//cdn.iframe.ly/api/iframe?url=https%3A%2F%2Fgithub.com%2Fgoogle%2Fclasp&amp;key=bc7b75ad5adf2dc500d4b0dee9f92949"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 気をつけたところ
あまりコード書かない方でも使えるように、なるべく簡単に開始準備が出来るように気をつけた。具体的には以下の点を考慮している。

- GASのコードは編集せずにスクリプトのプロパティをするだけで有効化出来る
- READMEを読めば一通りの設定が終わる

本当は認可周りの設定もコマンドで完了したかったのだが、プロジェクトIDなどの取得のためにどうしてもWebブラウザを開くことは回避できないのでひとまず諦めた。
また、現状の`clasp`コマンドではスクリプトプロパティの更新などは出来ないはずなので、こちらも諦めた。

# 今後
自動集計する仕組みは作ったものの、まだそのデータを使って可視化や分析は出来ていない。グラフなどを作ったらそのTipsも公開できたらよいと思っている。
また、他にも集計したほうがよさそうな項目があれば適宜追加していきたい。その週の記事投稿数なども必要かなと思っている。
そもそもTypeScriptで書いておきつつ型付けができていないなどの問題もあるのだが、Analytics APIがnpm上に存在しない（？）などもあるのでそこの解決はあまり優先していない。

# 最後に作ってみて
有志に人柱を依頼したのだが、好評をいただけたりして良かった。[@mom0tomo](https://twitter.com/mom0tomo)さんや[@ngmt83](https://twitter.com/ngmt83)さんにはすでにPRも作っていただいて不足点を改善していただいた。
この調子で今年は誰かに使ってもらえるOSSをいくつか作っていく（出来たらGoで作りたいな）。

# 参考
- https://github.com/budougumi0617/blog-kpi-collector

