+++
title= "Redash Meetup #0.1 - Redash 初心者向けハンズオン #redashmeetup"
date= 2018-01-31T14:06:34+09:00
draft = false
slug = ""
categories = ["report"]
tags = ["redash","handson"]
author = "budougumi0617"
+++


Redash Meetupに参加して、Redashについて学んできた。

|||
|---|---|
|URL|https://redash-meetup.connpass.com/event/75423/|
|会場|株式会社ココラブル|
|日時|2018/01/30(火) 19:00 〜 22:00|
|Toggeter| https://togetter.com/li/1194466 |
|ハッシュタグ| [#redashmeetup](https://twitter.com/search?q=%23redashmeetup) |


# TL;DR
- Docker環境さえあればすぐにハンズオンを開始できる。
  - https://github.com/kakakakakku/redash-hands-on
- 基本的な操作を2時間で学習することが出来た！
  - クエリを作ってグラフを作る
  - グラフをダッシュボードに貼る
  - しきい値を作って外部サービス（Slack）に通知を流す
- ちょっとした便利な使い方、Tipsも教えていただけた
- ハンズオンリポジトリはSQLの勉強にも使えるかもしれない？？
  - データソースにサンプルデータが山ほど入っているので、いろいろクエリが作れる
  - Redashのクエリ編集画面は補完も効くし、構文エラーもすぐわかる

# Redashとは
https://redash.io/

RedashはダッシュボードツールでBigQuery、Google Spreadsheetなど様々なデータソースの可視化を行えるOSS。


# ハンズオンに望む前の自分のRedashの理解
- Redashは自社でKPIをグラフ化するのに使っている。
- 構築やクエリを書いたのは他の人なので、自分はSlackに定期的に投稿されるRedashのグラフを見ているだけ
- 自分でもクエリを書いたりして使いこなせるようになりたいと思った。


# ハンズオン内容

ハンズオンは自分のPCにデータソースにもろもろ投入済みのRedashを起動して行われる。リポジトリは以下。
Redashのバージョンとしては2018/01/30時点ではv2.0とのこと。

https://github.com/kakakakakku/redash-hands-on

Docker環境さえあれば、`docker-compose`するだけで準備が完了するようになっている。
Docker for Macが入っている自分のMacではすぐに動かすことができた。
Docker for Windowsが使えないWindows 10 Pro以外のWindowsなどは（他に比べて）ちょっと準備がめんどくさいかもしれない。

```
$ docker-compose run --rm server create_db
$ docker-compose up
```

起動後は `http://localhost`にアクセスして、READMEに書かれた演習内容を進めていく。

https://github.com/kakakakakku/redash-hands-on/blob/master/README.md


## ダッシュボードを作るときは「グループ名:ダッシュボード名」という命名規則にする
http://help.redash.io/article/63-grouping-dashboards

命名規則を守っておくと、グルーピングされるとのこと。  
Redashを多数で使う場合、チームやプロダクトでダッシュボードをまとめられるのは非常に便利。

## クエリを探す時は3文字以上入力する
英語圏の事情なのかダッシュボード編集時などでクエリを探す時三文字以上打たないとサジェストされてこない。

## URLからクエリにパラメータを渡す
[パラメータ付きクエリを作ってみよう](https://github.com/kakakakakku/redash-hands-on/blob/master/README.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E4%BB%98%E3%81%8D%E3%82%AF%E3%82%A8%E3%83%AA%E3%82%92%E4%BD%9C%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%88%E3%81%86)

パラメータ付きのクエリを作っておけば、URLをちょっと変更するだけで個別のクエリができる。  
また、ドロップダウンリストにパラメータ候補を用意することもできるので、SQLがわからない人でも個別に見たい結果を作れる。

他にも[クエリの結果に色をつけたり](https://github.com/kakakakakku/redash-hands-on/blob/master/README.md#%E3%82%AF%E3%82%A8%E3%83%AA%E7%B5%90%E6%9E%9C%E3%81%AB%E8%89%B2%E3%82%92%E4%BB%98%E3%81%91%E3%82%88%E3%81%86)、
[アラートの設定をしたり](https://github.com/kakakakakku/redash-hands-on/blob/master/README.md#%E3%82%A2%E3%83%A9%E3%83%BC%E3%83%88%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%97%E3%82%88%E3%81%86)、
[独自スニペットを登録したり](https://github.com/kakakakakku/redash-hands-on/blob/master/README.md#%E3%82%AF%E3%82%A8%E3%83%AA%E3%82%B9%E3%83%8B%E3%83%9A%E3%83%83%E3%83%88%E3%82%92%E6%B4%BB%E7%94%A8%E3%81%97%E3%82%88%E3%81%86)、
2時間でデータソースからグラフの作成、通知まで一通りのことを学ぶことができた。

自分はSQLがあまりわからないので少し質問したが、SQLがわかる人ならREADMEの内容だけで最後まで学習できていた。


# ハンズオンを終えて
今まで会社のRedashは設定済みのグラフを見るだけだったが、
「ちょっとクエリ書いて自分の数値目標のグラフと通知作ってみよ！」と思えるようになれてすごいよかった。  
こんなこと言うと参加者が減ってしまって迷惑かもしれないが、ハンズオン内容がしっかりREADME上に書かれているので、
独りで自己学習でも十分最後出来ると思う。  
また、自分はSQLが全然書けないのだが、ハンズオンのリポジトリを今後はSQLの練習として使えるかなと思った。  
Redashのクエリエディタはサジェストが非常に優秀だし、
SQLの勉強する時、ある程度のデータがないと書けるクエリに幅がでない。  
このハンズオン資料は相当量のエントリが準備されているので、いろいろいクエリを書くネタにできそうだと思った。

あと、ちょっとハンズオンとは関係ないけれど、自分は今年はアウトプット駆動学習を目標にしているので[@kakakakakku](https://twitter.com/kakakakakku)さんに会えたのも良かった。

- [アウトプット駆動学習を習慣化する](http://kakakakakku.hatenablog.com/entry/2017/12/22/173455)

