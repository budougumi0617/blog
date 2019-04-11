+++
title= "golang.tokyoの技術書典6新刊「文Go」に「Goにおけるデータベース実践入門を寄稿しました。 #技術書典"
date= 2019-04-11T18:58:21+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["Go", "sql", "db", "技術書典"]
keywords = ["技術書典", "技術書典6", "Go", "golang", "db", "database/sql"]
twitterImage = "2019/04/11_bungo.png"
+++


2019/04/16に行われる[技術書典6](https://techbookfest.org/event/tbf06)へ、golang.tokyoも参加します。  
私は、今回の新刊である「文Go」に「Goにおけるデータベース実践入門」という内容で寄稿しました。

![表紙](/2019/04/11_bungo_s.png)

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://techbookfest.org/event/tbf06/circle/63860004" data-iframely-url="//cdn.iframe.ly/DxmrlWl"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# golang.tokyo 技術書典6 新刊「文Go」について
文Goは私以外にも6名の方々が執筆しており、以下のような内容になっています（敬称略）。

- 技術書典6 golang.tokyo サークル詳細
  - https://techbookfest.org/event/tbf06/circle/63860004


- 第1章 Go に関するよくある質問に答える [@tenntenn](https://twitter.com/tenntenn)
- 第2章 Go におけるデータベース実践入門 [@budougumi0617](https://twitter.com/budougumi0617)
- 第3章 GoとKafkaを用いた分散メッセージシステム入門 [@syossan27](https://twitter.com/syossan27)
- 第4章 VimでGoを快適に書く [@gorilla0513](https://twitter.com/gorilla0513)
- 第5章 Goによる画像処理のエッセンス [@po3rin](https://twitter.com/po3rin)
- 第6章 Goの画像合成デザインパターン [`@__timakin__`](https://twitter.com/__timakin__)
- 第7章 費用対効果の高いユニットテストを実現するためのGoの基礎技法 [@hgsgtk](https://twitter.com/hgsgtk)

すべての内容が技術書典6のための書き下ろしになっています。  
計136ページで販売価格500円なので、なかなかお買い得ではないでしょうか。
（なお、売上はすべてgolang.tokyoのノベルティ作成などに使われます）


# 寄稿内容について
私が寄稿した「Goにおけるデータベース実践入門」では、GoからRDBMSを操作する実装やテストの作成方法を紹介します。  
2019年になり、Goの日本語書籍は多く発売されています。また、インターネット上の日本語記事も多くなりました。
しかし、データベースを使った実装、テスト方法の基本的な部分をまとめた入門用の文章はほとんどないかと思います。
そこで今までGoのコードを使ってデータベースを操作する実装・テストを行なったことがない方向けに、次の情報をまとめました。  

- Dockerを利用したデータベースの準備方法
- `github.com/rubenv/sql-migrate`を使ったデータベースのマイグレーション方法
- リポジトリパターンを使ったデータベースアクセスと業務ロジックを疎にする設計方針
- `github.com/DATA-DOG/go-sqlmock`を使った単体テストの書き方
- `database/sql`パッケージを使ったデータベース操作の入門
- MySQLサーバを利用した結合テストの書き方
- CircleCI上でMySQLサーバを利用した自動テストを行なう設定例

プロダクションで動かしたさいの知見などをまとめたわけではないで基本的なことばかりですが、以下のような気持ちのビギナーの方にオススメです。

- GoでDB操作したいけどどこかに情報まとまってない？
- DB操作するコードってどうやってテストコード書けばいいの？

当日は販売も担当します。「技術書典6 け27 golang.tokyo」でお待ちしております。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://techbookfest.org/event/tbf06/circle/63860004" data-iframely-url="//cdn.iframe.ly/DxmrlWl"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://techbookfest.org/event/tbf06" data-iframely-url="//cdn.iframe.ly/ZYl8KPm?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
