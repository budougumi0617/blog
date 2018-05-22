+++
title= "[書評]SQL 第2版 ゼロからはじめるデータベース操作 を読んだ"
date= 2018-05-23T22:27:30+09:00
draft = false
slug = ""
categories = ["review"]
tags = ["book","sql"]
author = "budougumi0617"
+++



今更ながら（？）表題の本を読んだので読書メモを書いておく。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4798144452&linkId=4cdc008d82323946e1899eb0827ab4b5"></iframe>

# 所感
## SQLの知識がまったくない人から読者の対象
この本の読者の対象レベルはタイトルの通りまったくのSQL未経験者を対象としている。  
自分のレベルはだいたいこれくらいだった。

- 基本情報技術者レベルで単語の意味は知っている
- なんとなくActiveRecordやLINQは使えるが、対応するSQLが書けない
- ググりながらDBサーバを起動してツールやCLIから接続することはできる

「本書の対象となる読者」に記載されている例からすると以下に当てはまる状態だった。

- データベースを使うことになったものの、何から手をつければ良いかわからない人

ちなみに「本書で学習するための前提知識」にはWinodws関連のことが記載されているが、基本Macで問題ない。  
9章にあるJavaアプリケーションからの接続までちゃんとやるならばWindowsでやったほうが良いかもしれない。

## 本で学習できる範囲
本書では主に以下の内容を学習することができる。  
基本的にプロジェクトに参加してデータベースを利用したり、障害対応でデータベースを調査するくらいならば十分ではないだろうか。

- PostgreSQLを利用した学習環境の構築
- データベースの基本概念
- テーブルの作成/削除
- 基本的なデータ型や制約
- 検索クエリの作りかた
- 検索条件の指定や並び替え、グループ化
- データベースへデータを追加、更新、削除
- サブクエリやビューの利用
- 関数や述語
- テーブル同士の集合操作
- ウインドウ関数を使った高度（？）な操作
- Javaのアプリからデータベースへ接続・操作

ちなみに練習問題はほとんどないので、
もし「問題を解きながら理解を深めたい」というタイプならば別の本も併用したほうがよいと思う。  
自分の場合はMac(Docker)でMySQLのデータベース環境を作成してクエリを投げたりしていた。  
Dockerで練習環境を作るやり方は以下に記載してある。  

[Dockerで使い捨てのMySQL環境を用意する。事前データを投入して起動する。](/2018/05/20/create-instant-mysql-by-docker/)

## チューニングやテーブル設計については学べない
DBのパフォーマンスチューニングついての話、たとえばスロークエリを作成しないための方法やindexの効果検証などの方法は載っていない。  
また、テーブルのスキーマの設計などに関しての言及もない。  
とはいえ、読者対象を考えるとデータベースの検索と基本操作が出来るようになる内容で十分であると思う。

## 終わりに
SQLをまったく触っていない自分でも一通りの検索クエリを書いたり読めるようになった。  
ただ、まだ最適なクエリ（JOINでやるかサブクエリでやるかetc）を書くには次の本を読む必要がある。  
自分は[SQLパフォーマンス詳解](https://amzn.to/2KPFONh)を読む予定。  
PDFを買ったのだが、インターネット上でも読めるようだ。

**SQLのインデックスとそのチューニングについてのオンラインブック**  
https://use-the-index-luke.com/ja

# 関連
- [Dockerで使い捨てのMySQL環境を用意する。事前データを投入して起動する。](/2018/05/20/create-instant-mysql-by-docker/)
- [SQLのインデックスとそのチューニングについてのオンラインブック](https://use-the-index-luke.com/ja)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4798144452&linkId=4cdc008d82323946e1899eb0827ab4b5"></iframe>
