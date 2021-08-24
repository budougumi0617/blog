+++
title= "ISUCON11予選に参加した（最終スコア47,995点 79位） #isucon"
date= 2021-08-24T09:55:49+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["isucon"]
tags = ["iscuon","isucon11"]
keywords = ["isucon","newrelic","netdata","alp"]
twitterImage = "twittercard.png"
+++

ISUCON11予選にSpeed of Soundというチームで参加した。最終スコア47,995点で予選敗退だった。  
当日のメモややっていたよかったこと、次回に向けて頑張りたいところなどまとめておく。

<!--more-->

# TL;DR
- 事前に準備したこと
- 当日のメモ
- よかったこと
- だめだったところ
- 次回に向けてがんばりたいところ

当日の作業リポジトリはこちら
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/isucon11q" data-iframely-url="//cdn.iframe.ly/4J9jRr2?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## ISUCON11
説明する必要もないかもしれないが、ISUCONとはお題となるWebサービスを決められたレギュレーションの中で限界まで高速化を図る競技のこと。  
詳細や魅力については次のリンクの記事を参照のこと。

- 寿命が縮まる、人生が変わる…８時間耐久でインフラ技術競い合うISUCONの魅力とは？過去優勝者・主催者に聞く
    - https://type.jp/et/feature/13831/

今年は11回目のISUCONが行われ、私ははじめて参加した。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://isucon.net/archives/55821036.html" data-iframely-url="//cdn.iframe.ly/dvPCvIN?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

予選の問題のリポジトリとマニュアルは次のリンクの通り。

- ISUCON11予選問題
    - https://github.com/isucon/isucon11-qualify
- 予選当日マニュアル
    - https://gist.github.com/ockie1729/53589a0e8c979198b6231d8599153c70



## 事前に準備したこと
参加するにあたり、次の2点を解決するために環境構築の練習やスクリプトの作成をやっておいた。

- 初期環境構築で躓いて終わるのは嫌だった
- 業務ではAWSのマネージドサービスを使い自分が触る部分はほぼコンテナなので、「Linuxサーバ」や非マネージドのミドルウェアの扱いに詳しくない

練習では[aws-isucon](https://github.com/matsuu/aws-isucon)のisucon10qのAMIを使わせていただいた。  
練習した自分用メモやスクリプト、スニペットは次のリポジトリにまとめておいた。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/isucon-cheatsheet" data-iframely-url="//cdn.iframe.ly/ep9Zgqx?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

また、業務ではNew Relic Oneを利用しているため、ISUCONでもNew Relicを含めた以下のツールをすぐ導入できるように手順をまとめていた。

- New Relic One
- Netdata
- pt-query-digest
- alp

本当ならば実際のチューニングも練習したかったが、いろいろな事情でisucon10の講評を見ながら少しインデックス貼るくらいしか準備できなかった。


## 当日の作業記録
再掲になるが、当日の作業リポジトリは以下。

- https://github.com/budougumi0617/isucon11q

午後に入ってから雑になっているが、当日は以下のissueで進捗メモをしていた。

https://github.com/budougumi0617/isucon11q/issues/1

自分の当日の行動は次のような感じだった。

- 9:40 オープニング視聴
- 10:11 CloudFormationで環境構築完了。フロントエンドにアクセスできるのを確認
- 10:30 git initやひととおりセットアップするスクリプトなどを流し終わる
    - [github.com/budougumi0617/isucon-cheatsheet/script/tool_setup.sh](https://github.com/budougumi0617/isucon-cheatsheet/blob/d642a10057342866d73af615a72b086720022cb8/script/tool_setup.sh)
- 10:50 New Relic OneでサーバとMySQLのメトリクスを見れるようになる
- 10:55 あれ？MariaDBなの？って気づく。mariadbサービスもmysqlサービスも動いていて、systemctlとかどうなってるのかわからず混乱してくる。
- 11:23 シンボリックリンクにしたmy.cnf上のスロークエリの設定が反映されなくてシンボリックリンク運用を諦める。
- 11:40 alpでいい感じにまとめる正規表現を設定ファイルに書く。スロークエリが集計できることを確認
- このへんからindexとかはってもらう
- 13:00Newrelic APM入れようと試すも分散トレースがうまくうごかず。
    - v4リリースタグがついているライブラリのgo get方法がわからずlogs in contextも諦める。
- このへんでスコアはだいたい1,100点くらい。
- 15:00 まだNewrelicの分散トレースためしていたけどやっぱり動かず。完全に諦める
- 1台目のサーバからDBを2台目のサーバに移す作業をする
- スロークエリみたりアクセスログみたいするけど
- 17:00 今更DBに画像入っていることに気づく。抜き出そうと思うがいろいろ知見不足であきらめる。
- 17:30 もろもろの監視設定をオフに。
- 延長時間中はトランザクションいらなそうなところ外したり。再起動試験で0点にならないよう大きな変更は控えていた。

## よかったこと
### セットアップは順調にできた
事前に準備していたスクリプトや素振りのおかげでNew Relic Infrastructureの導入まではすんなり準備できた。
環境情報収集を収集してissueに書き出す流れも組んでいたのでMariaDBの存在や動いているサービスをすぐ把握できた。

- https://github.com/budougumi0617/isucon11q/issues/1#issuecomment-903031446

### DB分離はすんなりできた
CPU負荷がapp/nginx/dbすべてでそこそこ高かった。どうせ3台あるので早々に2台目をDBサーバにした。
`bind-address`などにハマることなくすんなり分割できたのはよかった。

### 変なところに力を入れずにできた
いろいろあれば便利なのだろうが準備不足だったのでコスパよく工夫できたかなと思った。
2人で参加したのでデプロイは声掛けだけでやった。基本はローカルPCで修正・PR作成だったが、試行錯誤するときはサーバ上で直接コードやSQLを編集した。
業務ではないので、「8時間で結果を出す」という目的に対して柔軟にできたかなと思う。
ビルドスクリプトはログファイルtruncateしてサービスをリスタートするだけのMakefileをさっと書いた。

```makefile
reset:
	sudo truncate --size 0 /var/log/nginx/access.log
	sudo truncate --size 0 /var/log/mysql/slow.log
	go build -o ./isucondition
	sudo systemctl restart isucondition.go.service

alplog:
	sudo cat /var/log/nginx/access.log | alp -c /home/isucon/webapp/alp.yml ltsv > ~/alp.log

reset-slowlog:
	ssh isu11B truncate --size 0 /var/log/mysql/slow.log
```

`/etc`配下の設定ファイルもシンボリックリンクでgit管理するつもりだったがうまく設定が当たらなかったので早々に諦めた。
この辺も目的ベースで判断できたかなと思う。

### ghコマンドのissueコメント機能が便利だった

記録に関してはSlackなどに流せるとスマートだったのだろうが、シンプルに`gh`コマンドでissueにコメントしていくスクリプトを使った。
リモートのログファイルもissueにメモしておけたりとても便利だった。

https://github.com/budougumi0617/isucon11q/blob/master/script/gh_push_memo.sh

```bash  
#!/bin/bash -eu

# ./gh_push_memo.sh 1 "`ssh isu10A cat /home/isucon/slow_dgst_20210731_151419.log`"
# って感じで使うと第一引数の番号のissueに第二引数の結果をコメントしてくれる。実行はgit repo内でやる（-Rで指定してもいいが…）
BODY="
\`\`\`
$2
\`\`\`
"
gh issue comment $1 -b "$BODY"
```

## だめだったところ
### Newrelic APM+log in contextでドハマリした
「分析力不足をNewrelicのフルスタックオブザーバビリティで補おう！」と思ってNew Relicをフルに導入しようと思ったが、失敗した。
うまくtraceが動かず、logs in contextもライブラリがうまく入らず導入できなかった。
並行作業をしながらの導入だったものの15時くらいまで時間を溶かしてしまってこれに関しては1時前の段階で諦めるべきだった。

### 全然パフォーマンス改善に寄与できなかった
上記でハマっていたこともあって自分でアプリやSQLの改善をしたのはちょっとindexを貼ったくらいだった。
また、マニュアルもほとんど読まなかったので「このエンドポイントを非同期化する！」みたいな考えができていなかった。

### 分析に対してアプローチできなかった
「計測結果が悪いけど直し方がわからん」というときと、（手数増やすと言う意味ではいいんだけど）「たぶん良くなるからいじってみる」ということが多く、実力不足を痛感した。  
スロークエリの降順INDEX貼れば良さそうなログも「ISUCON10予選にもあったけれど、timestampの負数とは。。。？」みたいになり何もできなかった。
MariaDBの知識不足で不安になるならガッとMySQLに変える腕力があってもよかった。
その結果、「なんとなく悪そうだからここ直せばスコアあがるだろ」みたいなトライアンドエラーが多くなってしまい、後半止まってしまった時間もあった。  
一方で「`SELECT`結果から不要に取得しているBLOBを消しておく」みたいな堅実な対応もできていなかった。

[最近読んだ「早く正しく決める技術」](https://amzn.to/384Zavi)には次のような指摘があった。

> 結果の検証が十分できないというのは、往々にして計画が甘いことが原因です。
> 実行自体はいいとして、計画が甘いからチェックができず、したがって改善策もなかなか出てきません。

結局こうやって振り返りしていても目的をもって行動していないアクションに対してはあまり改善や課題も浮かばず、もっと意思や決断をもってチューニングできないとなと思った。

### ちゃんと振り返りをするためにもう少していねいに記録をとっておくべきだった。
マージのたびにベンチマーク実行、実行結果のURLをメモしていたけれど、終わったあとは見れなかった。
最後にベンチマーク実行履歴一覧のスクショを撮っておくべきだった。

## 次回に向けてがんばりたいところ
nginx/RDB/Go どのレイヤーも勉強が必要…。

## 最後に
普段の業務はどちらかというと「マネージドサービスをどれだけ使い倒せるか？」という志向であり「生のLinuxサーバ上でのアプリ構築が業務に直結するか？」っていうと微妙なところもあるかもしれないが、もろもろの考え方やアプローチ（あと単純な腕力とか瞬発力とかもね）は地続きでつながっていると思うので今回の悔しさをバネに勉強したいと思った。    
今回結構前から会社のSlackにISUCONチャンネルを作ったのだが、同僚にISUCON仲間が何名できたしモチベーション維持して早い段階から特訓していきたい！（とくに[@cureseven](https://twitter.com/cureseven)さんには毎回練習に誘っていただき感謝しかない）。  
実は来年も継続開催してほしい気持ちで個人スポンサーもやっていた。ISUCON12目指して少しずつスキルアップしていく！  
（本戦出場のみなさんは本戦でもがんばってください！）

# 参考
- ISUCON11まとめ
    - https://isucon.net/archives/55821036.html
- https://github.com/isucon/isucon11-qualify
- 予選当日マニュアル
    - https://gist.github.com/ockie1729/53589a0e8c979198b6231d8599153c70
- https://github.com/matsuu/aws-isucon