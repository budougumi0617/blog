+++
title= "[書評] 改定2版 みんなのGo言語はGo言語入門者にも初版所有者にもオススメな1冊"
date= 2019-08-04T15:22:15+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["review", "go"]
tags = ["golang", "book"]
keywords = ["Go", "みんGo", "書評", "みんなのGo言語", "改訂2版 みんなのGo言語"]
twitterImage = "2019/08/04_ogp.jpeg"
+++

[@tenntenn](https://twitter.com/tenntenn)さんに献本していただいたので、改訂版2版みんなのGo言語（通称みんGo）の感想をまとめる。  
なお、私は初版も持っているので、「前の持っているしどうしよう？」という方向けに初版との比較も記載する。

![表紙画像](/2019/08/04_mingo.jpg)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07VPSXF6N&linkId=eeb24badc64c61fb48808b960c329c80"></iframe>

<!--more-->

# 所感
端的に言うとGoの開発をするときPCの隣に置いておきたい1冊だ。  
言語仕様を読んだり[Tour of Go](https://tour.golang.org/)をするだけではわからないTipsやベストプラクティスがふんだんにまとまっている。  
「今年はGoやるぞ！環境構築してCLIツールかTODOアプリのWEBページでもテスト書きながら作ってみよう。」という人はこの一冊があれば十分だろう。


初版を持っている方でも、もう一度買って十分得する本だと思う。私は持ち歩かなくてもよいように電子書籍版も購入した（初版の物理本・電子書籍も合わせると4冊？所有していることになる）。  
後述するが、改定2版では主に以下の点が更新されており、ページで言うと初版から20ページほど増量されている。

- Go1.7以降に追加された仕様への言及
- 2016年から2019年の間に移り変わったOSSトレンドの追従
- Go Modulesの使い方
- `database/sql`を使ったデータベース操作
- 簡単なWebアプリケーションの実装サンプル

# どんな本なのか
本書は2016年に発売されたみんなのGo言語の改定版である。1版は16年9月に発売されたのでほぼ3年経っての改定である。  
初版のころからGoを今から始める人に非常にオススメの本で、この本1冊で以下のようなコトがわかる。

- [motemen/ghq](https://github.com/motemen/ghq)などを使った環境構築方法やGopher（Go利用者）の多数が利用しているであろうインストールしておきたい便利な開発ツール
- マルチプラットフォーム、サブコマンドやオプション対応を実装した実践的なCLIツールの作成方法
- バッファリングやgoroutine処理、リフレクションなどの一工夫必要な実装の解説
- Racce Detectorや`TestMain`関数を使った一連のGoのテストの書き方
- `database/sql`を使ったデータベース操作と簡単なWebアプリケーションの作成方法（改定2版で追加）

本書は網羅的に言語仕様をカバーする類の本ではないが、OSSのコード・業務コードを読んだときに「何やっているんだろう？（なぜこうする必要があるんだろう？）」と疑問に思うようなことはほとんど押さえてあるだろう。  
日頃Goの情報にアンテナを張っているような人でないと知らない内容もあったりするので、初級者以上でも知らないことがないか読んだほうが良いだろう。

# 初版と改訂2版の違いについて
初版と比較して改定2版では以下の更新・追加が行われた。

## Go1.7以降に追加された仕様への言及
2016年に発売された初版当時のGoのバージョンは1.6だった。そのため、初版発売以降に追加されたGoの仕様に対する言及が各ページに追加されている。

- `1.3 Goを始める` Go1.11から試験導入されているGo Modulesについて
- `2.8 設定ファイルの取り扱い` Go1.12から追加された`os.UserHomeDir`関数について
- `3.8 goroutineの停止` Go1.7から追加された`context.Context`を使った外部からのgoroutineの停止
- `6.2 testingパッケージ入門` Go1.10で追加されたテストキャッシュについて
- etc...

## 2016年から2019年の間に移り変わったOSSトレンドの追従
どの言語にもデファクトスタンダードになっているOSSがあるだろう。本書でも多数のOSSが紹介されている。  
改定2版では新しいOSSの紹介や3年間で移り変わってトレンドが反映されている。

- `1.2 エディタと開発環境` 新進気鋭の公式LSPの`gopls`の紹介
- `2.6 シングルバイナリにこだわる` 初版で紹介されていたが、メンテがされなくなった[jteeuwen/go-bindata](https://github.com/jteeuwen/go-bindata)ではなく、[rakyll/statik](https://github.com/rakyll/statik)に変更

## Go Modulesの使い方
2019年現在、Goの依存パッケージ管理といえばGo1.11から試験導入され始めたGo Modulesだろう。Go1.13では`go.mod`ファイルさえ置いておけば自動で有効になる。

- tip Go 1.13 リリースノート
    - https://tip.golang.org/doc/go1.13#modules

初版では`glide`を使ったパッケージ管理方法が紹介されていたが、改定2版で`go mod`を使ったGo Modulesに置き換わっている。  
（雑誌を除く）日本語の書籍でGo Modulesを使った方法を言及している書籍ははじめてではないだろうか。

## database/sqlを使ったデータベース操作と簡単なWebアプリケーションの実装サンプル
改定2版で追加された第7章ではデータベース操作の基本が説明されている。
7章も大変実践的で、単純な公式パッケージの説明だけでなく、以下のような業務で使うときに一度は悩むことが丁寧に解説されている。

- `sql.NullString`型を使いつつJSON Marshalingに対するためのTipsや対応しているOSSの紹介
- ORMを使ったデータベース操作
- データベースを操作するWEBアプリケーションの作成

初版ではWEBアプリケーションを作るような章はなかった。  
この1冊でCLIツールとWEBアプリケーションの作成を試せるようになったという点でも7章が追加された意味は大きいと思う。

# 少し気になったこと
全体的に大満足なのだが、気になるところがなかったわけではないので、その点も記しておく。

## sync.Mapの紹介があってもよかったかも?
`1.4 Goらしいコードを書く`の中の`mapを避ける`の節では、スレッドセーフではない`map`を使うために`sync.RWMutex`を利用したマップの実装が記載されている。  
排他制御の実装例として良いと思うのだが、Go1.9からは`sync.Map`型が追加されているので、`sync.Map`への言及があってもよいと思う。

## 表記が章ごとにまちまち
書籍の中では全体的に関数名や変数などのキーワードは太字の文字装飾がされている。一部の関数名が太字になっていなかったところがあった。  
また、本書は章ごとに著者が異なる本である。章によっては**Setup**`関数`、**io.Writer**`型`のように、太字のキーワードが関数なのか？型なのか？などまでしっかり記載されている。  
初心者ほど名前を見ただけでは関数なのか変数なのかわからないので、全部の章でキーワードの種類も書いてあるともっと読みやすくなるのでは？と思った。

# 終わりに
主に改定された内容についての言及が多くなってしまったが、改定2版 みんなのGo言語について感想をまとめた。  
個人的な話をすると献本したいただいたのははじめてだったので、そういった意味でも私に大事な１冊となりそうだ。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07VPSXF6N&linkId=eeb24badc64c61fb48808b960c329c80"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=5f0980cada3e4a75907a75ae671438ea"></iframe>

