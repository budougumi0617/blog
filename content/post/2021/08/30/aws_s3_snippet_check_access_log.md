+++
title= "S3に保存されたAWS ALBのアクセスログをローカルでさっと確認するワンライナー"
date= 2021-08-30T00:24:25+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["aws"]
tags = ["aws","ALB","log"]
keywords = ["aws","s3","alb","aws cli"]
twitterImage = "logos/aws.png"
+++

AWS ALBのログを漁って疎通確認をするときにちょっとラクするためのスニペットメモ。

```bash
$ aws s3 ls s3://${ALB_LOG_BUCKET}/AWSLogs/${ACCOUNT_ID}/elasticloadbalancing/ap-northeast-1/2021/08/30/ \
| awk '{print " s3://${ALB_LOG_BUCKET}/AWSLogs/${ACCOUNT_ID}/elasticloadbalancing/ap-northeast-1/2021/08/30/"$4}' \
| xargs -IS3URL aws s3 cp S3URL - | zcat
```

<!--more-->

# TL;DR
- ALBのアクセスログはS3バケットに圧縮ファイルとして保存される
- ログを確認したいとき、以下の手順を毎回踏むのがめんどくさい
    - AWS ConsoleからS3のアクセスログ用バケットを開く
    - オブジェクトの保存時刻を見てログが入ってそうな圧縮ファイルをダウンロードする
    - ローカルで解凍する
    - ログを確認する
    - 圧縮ファイルを削除する
    - 期待するログがなかったら再度圧縮ファイルのダウンロードから始める
- 以下のコマンドを実行すれば圧縮ファイルをダウンロードしなくてもログを確認することができる
- 該当日のすべての圧縮ファイルをGETするため、本番環境の利用時は課金など注意すること

```bash
$ aws s3 ls s3://${ALB_LOG_BUCKET}/AWSLogs/${ACCOUNT_ID}/elasticloadbalancing/ap-northeast-1/2021/08/30/ \
| awk '{print " s3://${ALB_LOG_BUCKET}/AWSLogs/${ACCOUNT_ID}/elasticloadbalancing/ap-northeast-1/2021/08/30/"$4}' \
| xargs -IS3URL aws s3 cp S3URL - | zcat
```


# AWS ALBのログを漁りたい
開発中にインフラ構築をしていたり外部との疎通確認にアクセスログを確認したいときが度々ある。
AWS ALBのアクセスログの取得は結構簡単で、次のドキュメント通りに設定すればよい。

- Access logs for your Application Load Balancer
    - https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html

ただし、設定は簡単なのだがS3に圧縮ファイルが保存されるので実際にログを探すのはやや骨が折れる。

## AWSコンソールからのログの確認手順
S3に保存されたログを確認するには次のような手順を踏むことになるだろう。

1. AWS ConsoleからS3のアクセスログ用バケットを開く
1. オブジェクトの保存時刻を見てログが入ってそうな圧縮ファイルをダウンロードする
1. ローカルで解凍する
1. ログを確認する
1. 圧縮ファイルを削除する
1. 期待するログがなかったら再度圧縮ファイルのダウンロードから始める

毎回ファイルを探すのが非常にめんどくさい。
また、圧縮ファイルをローカルに保存する（本番環境のバケットの中身を保存する）のがルール的にまずい組織もあるだろう。


# awsコマンドを使ってログを標準出力に流すワンライナー
どうにかラクできないか？と思って書いたワンライナーが以下になる。

```bash
$ aws s3 ls s3://${ALB_LOG_BUCKET}/AWSLogs/${ACCOUNT_ID}/elasticloadbalancing/ap-northeast-1/2021/08/30/ \
| awk '{print " s3://${ALB_LOG_BUCKET}/AWSLogs/${ACCOUNT_ID}/elasticloadbalancing/ap-northeast-1/2021/08/30/"$4}' \
| xargs -IS3URL aws s3 cp S3URL - | zcat
```

## 利用するコマンド
パイプでつないで各々のコマンドを使う意図は次の通り。

- `aws s3 ls`
    - https://docs.aws.amazon.com/cli/latest/reference/s3/ls.html
    - プレフィックスに条件一致するオブジェクトの一覧を取得する
- `awk`
    - https://ja.wikipedia.org/wiki/AWK
    - プレフィックスを付け直して`aws s3 ls`コマンドの結果を一覧する
- `xargs`
    - https://linuxjm.osdn.jp/html/GNU_findutils/man1/xargs.1.html
    - オブジェクトそれぞれに再度`aws`コマンドを十個する
- `aws s3 cp`
    - https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html
    - 出力先を`-`にしてオブジェクトの中身を標準出力へ
- `zcat`
    - https://linuxjm.osdn.jp/html/GNU_gzip/man1/gzip.1.html
    - 解凍を挟んで`cat`する

`aws s3 ls`コマンドの結果として出力されるオブジェクト名には検索でしていしたプレフィックスがつかない。そのため、`awk`コマンドでプレフィックスを付けている。  
ALBのアクセスログの場合のプレフィックスは次の通り。


> bucket[/prefix]/AWSLogs/aws-account-id/elasticloadbalancing/region/yyyy/mm/dd/aws-account-id_elasticloadbalancing_region_load-balancer-id_end-time_ip-address_random-string.log.gz


あとはその完全パスのオブジェクト一覧にたいして`xargs`コマンドで`aws`コマンドを再度実行する。
`zcat`は圧縮ファイルの中身を解凍してから`cat`してくれる便利コマンド。


# 最後に（注意点）
アクセスログを漁るときは`zcat`と`zgrep`を多用するが今回も`zcat`にはお世話になった。  
注意点としてはS3はオブジェクトをGETするにもコストがかかるので、本場環境で多用しすぎると思わぬ課金にあうかもしれない。利用は自己責任で。
私自身はALB構築時のアクセスログ保存確認くらいのタイミング（`curl`数回叩いたあとログが出てるか確認する）でしか使っていない。


- AWS S3 pricing
    - https://aws.amazon.com/s3/pricing/


`awk`と`xargs`力を上げればかなり効率的にコマンドライン生活を送れそうなのだがなかなか手が回っていない。この辺は地味にエンジニア力の基礎として大事そう。

# 参考
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html
- https://docs.aws.amazon.com/cli/latest/reference/s3/ls.html
- https://ja.wikipedia.org/wiki/AWK
- https://linuxjm.osdn.jp/html/GNU_findutils/man1/xargs.1.html
- https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html
- https://linuxjm.osdn.jp/html/GNU_gzip/man1/gzip.1.html

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4274064069&linkId=74681e06eb26825cad23f4e8f50fbd46"></iframe>