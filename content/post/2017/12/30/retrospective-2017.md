+++
title= "2017年振り返り"
date= 2017-12-30T19:05:30+09:00
draft = false
slug = ""
categories = ["poem"]
tags = ["Retrospective"]
author = "budougumi0617"
+++

2017年を振り返ってみる。GitHubベースの振り返りは別記事にまとめた。

[2017年振り返り(GitHub編)](https://budougumi0617.github.io/2017/12/27/retrospective-on-github/)

# 転職した
人生で初めて転職した。  
新卒から6年勤めていた精密機器メーカー子会社からクラウド会計・人事労務ソフトのWeb系ベンチャーに転職した。  
2016年に30歳を迎えたのだが30代をどうやって生きるか考えたとき、平日の大半の時間を占める就業時間中にもっとコードを書くことを選択したかった。  
前職最後の仕事が`C#`のデスクトップアプリ開発だった縁で現在の会社に拾っていただくことができた。  
ポジションとしてはデスクトップアプリエンジニアなのだが、クラウド側のAPI部分についても開発に携わらせてもらっている。  
Railsのコードも書いたり、WindowsストアにDesktop Bridgeアプリを公開したり、`Xamarin.Mac`によるデスクトップアプリの開発もすることができた。  
`Xamarin.Mac`はまだあまり事例などが少なかったり、`mono`フレームワークのバージョンが上がるまでWindows上で実行したときと結果が異なったりという苦労もあったが、
転職早々非常にアグレッシブなテーマにアサインしてもらうことが出来て非常に良い経験になった。  
ただ、求められているパフォーマンスは全然出せなかった。

- 「ドメイン知識の貯金」でやり過ごしていた
- 単純にコーディングの生産性が低い


# OSS活動
エンジニアとしてのPublicな活動を増やしていきたいと思い、OSSライブラリを作った。

https://www.nuget.org/packages/budougumi0617.Testable  

また、タイポ修正くらいしか出来ていないのだが、OSSにもプルリクを送り始めた。`golang`以下のリポジトリにマージされたのはかなり嬉しかった。
来年はちゃんとコードの修正でプルリク作りたい。


# ブログを書き始めた
もともとTipsはQiitaに投稿していたが、6月からは自分のブログに投稿することにして、このブログを作った(コミット量稼ぎも兼ねている)  
18記事なので、一月平均3記事程度だろうか。まだまだ全然少ない。「来年について」に後述するが、ここはもっとアウトプットを増やしていきたい。


# オンライン勉強会(AOSN読書会)
前職のつながりで[AOSN読書会](https://aosn.ws/)という週一開催のオンライン勉強会に参加し始めた。
読書会では、「プログラミングElixir」、「人月の神話」、「現場で役立つシステム設計の原則」を読んでいる。

- 同じ練習問題に対する異なる実装パターンを解説付きで知ることができる。
- 職種・職場が異なるメンバーと言語や技術についてディスカッションできる
- 自分はあまり勉強癖がないので、強制的な勉強する時間を作りたかった

今年から初めてスクリプト言語や関数型言語プログラミングに挑戦しているので、他の人に教えてもらいながらElixirの勉強が出来るのは心強い。

# 来年やりたいこと
来年はもっとアウトプットを意識したて活動をしていきたい。

[アウトプット駆動学習を習慣化する](http://kakakakakku.hatenablog.com/entry/2017/12/22/173455)

# 参加した技術イベント
- [golang.tokyo #3](https://golangtokyo.connpass.com/event/47328/)
- [golang.tokyo #5](https://golangtokyo.connpass.com/event/53209/)
- [JJUG CCC 2017 SPRING](http://www.java-users.jp/ccc2017spring/)
- [de:code 2017](https://www.microsoft.com/ja-jp/events/decode/2017/)
- [.NET Standardもくもく会](https://jxug.connpass.com/event/61044/)
- [グレープシティ ECHO Tokyo 2017](http://www.kokuchpro.com/event/echo2017/)
- [Xamarin.Forms本 邦訳読書会 & コードリーディング .823](https://xamarinformsbookreading.connpass.com/event/64546/)
- [Go Conference 2017 Autumn](https://gocon.connpass.com/event/66615/)
  - [Go Conference 2017 Autumn参加メモ #gocon](https://budougumi0617.github.io/2017/11/09/gocon2017-autumn/)
- [golang.tokyo #11](https://golangtokyo.connpass.com/event/72986/)
  - [[Go]golang.tokyo #11 参加メモ #golangtokyo](https://budougumi0617.github.io/2017/12/12/golangtokyo-11/)
- [Kubernetes ときどき Serverless -- cndjp第1回](https://cnd.connpass.com/event/69971/)
  - [Cloud Native Developers JP 第1回勉強会 参加メモ #cndjp1](https://budougumi0617.github.io/2017/11/23/cndjp1/)
- [Kubernetes in プロダクション ! -- cndjp第2回](https://cnd.connpass.com/event/73694/)
  - [[k8s]Cloud Native Developers JP 第2回勉強会 参加メモ #cndjp2](https://budougumi0617.github.io/2017/12/18/kubernetes-in-production-cndjp2/)

とくに意識はしていなかったが、ほとんどGoのイベントばかりだった。
冬からは意識してブログにレポートを残すことに決めた。ブログを書くまでがイベント参加。転職直後のキャッチアップを言い訳に控えていた時期も多かったので、来年はもっと参加していこうと思う。

# 2017年に読んだ書籍
- [C#実践開発手法　デザインパターンとSOLID原則によるアジャイルなコーディング](http://amzn.to/2EglT7D)
- [.NETのエンタープライズアプリケーションアーキテクチャ第2版　.NETを例にしたアプリケーション設計原則](http://amzn.to/2EfZnf5)
- [Effective C# 4.0](http://amzn.to/2ChLjUn)
- [プログラミングWindows第6版　上～C#とXAMLによるWindowsストアアプリ開発](http://amzn.to/2EgZHKJ)
- [C#逆引きレシピ［Advanced］](http://amzn.to/2EdLiie)
- [実戦で役立つ C#プログラミングのイディオム/定石&パターン](http://amzn.to/2Cndmlg)
- [たのしいRuby 第5版](http://amzn.to/2CnaXHm)
- [実践Ruby on Rails 4 現場のプロから学ぶ本格Webプログラミング](http://amzn.to/2q0dsdx)
- [実践Ruby on Rails 4 機能拡張編](http://amzn.to/2EgoMoZ)
- [Goプログラミング実践入門　標準ライブラリでゼロからWebアプリを作る impress top gearシリーズ](http://amzn.to/2CndvFk)
  - [[書評]Goプログラミング実践入門　標準ライブラリでゼロからWebアプリを作る](https://budougumi0617.github.io/2017/10/12/go-web-programing/)
- [Docker実践ガイド impress top gearシリーズ](http://amzn.to/2ltliqI)
- [人月の神話【新装版】](http://amzn.to/2lufixF)
- [SCRUM BOOT CAMP THE BOOK](http://amzn.to/2pXCg63)
- [日本企業がシリコンバレーのスピードを身につける方法](http://amzn.to/2pY0b53)
- [あなたのチームは、機能してますか？](http://amzn.to/2EdL4Yq)
- [キャズム Ver.2 増補改訂版 新商品をブレイクさせる「超」マーケティング理論](http://amzn.to/2EgSslM)
- [ジョイ・インク 役職も部署もない全員主役のマネジメント](http://amzn.to/2ClxRyA)
- [｢1日30分｣を続けなさい！Kindle版: 人生勝利の勉強法55](http://amzn.to/2EfOBFL)

.NET系の設計やアーキテクチャへの言及は流石MS本という内容で非常に勉強になった。そんなに`C#`のサンプルコードも難しくないので、`C#`使わない人でも読めると思う。
また、業務で使うということで、`Ruby`や``Ruby On Rails`の本を一通り読んだ。
書評を全然書いていなかったので、本に関しても来年は読後のまとめを作る。正直春先に読んだ本はだいぶ記憶がおぼろげだ…

# 2017年に読み終わらなかった書籍
- [プログラミングElixir](http://amzn.to/2ClBUuR)
- [Real World HTTP ―歴史とコードに学ぶインターネットとウェブ技術](http://amzn.to/2pV2Ufy)
- [プログラマのためのGoogle Cloud Platform入門 サービスの全体像からクラウドネイティブアプリケーション構築まで](http://amzn.to/2CndNfo)
- [Goならわかるシステムプログラミング](http://amzn.to/2lsKQEg)
- [データ分析基盤構築入門［Fluentd，Elasticsearch，Kibanaによるログ収集と可視化］](http://amzn.to/2CnoONw)
- [現場で役立つシステム設計の原則 〜変更を楽で安全にするオブジェクト指向の実践技法](http://amzn.to/2Cl39ph)

中途半端に読んでいる本が多いので、「読了するまで次の本は読まない」という制限が必要かもしれない。
あと、これの倍以上「積読」があるので読まないといけない。


# まとめ(KPT)
## Keep
- 初めて転職した
  - 転職がきっかけでWBSデビューしたりした
- `Destop Bridge`アプリを作成し、Windowsストアに公開した。
- `Xamarin.Mac`アプリを作成した
- 自作OSSを公開した。
- `golang/dep`などに自分の修正がマージされた

## Problem
- OSSにコーディングでプルリク出せるレベルではない
- 勉強時間の確保（習慣化）が出来ていない
- 生産性をもっと上げたい。
- 読み終わってない本が多い。

## Try
- もっとコードでOSSに貢献する
- 自分のOSSをもっと作る
- GoとGCPでWebアプリを作る
- クラウド・コンテナ技術の知識を身につける
- 朝の学習時間をもっと確保・習慣化する
- ブログをもっと更新する
