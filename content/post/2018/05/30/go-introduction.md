+++
title= "[発表資料]Go入門"
date= 2018-05-30T08:58:32+09:00
draft = false
slug = ""
categories = ["go", "presentation"]
tags = ["golang"]
author = "budougumi0617"
+++

Vue.js/React/Go/Rails5.2のリアル★ShuuuMai #04の発表資料と資料中の参考リンク、補足をまとめた。
https://connpass.com/event/86508/

<script async class="speakerdeck-embed" data-id="e6fbf358f82c4894b0546f467abb2e9f" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# Golang

https://golang.org/

Go自体のコードはここから読める。

https://github.com/golang/go

# Language Specification
Goの言語仕様については以下のページで詳細を確認できる。

https://golang.org/ref/spec

# Concurrent programing by Go
Goの平行処理についての詳細は以下が参考になる。

https://github.com/golang/go/wiki/LearnConcurrency

「並行」処理と「並列」処理の違いは以下の発表を読んでみるとよい。

**Concurrency is not Parallelism**  
https://talks.golang.org/2012/waza.slide#1  
https://vimeo.com/49718712

# GoとOSS
以下のようにGoで作られたOSSは多い。

**プログラミング言語 - Goの記事一覧**  
https://www.moongift.jp/tags/go

# Webでカジュアルに試す
Tour of Goならば環境構築をせずに、オンラインでGoの言語仕様を一通り学習できる。

**A Tour of Go**  
https://tour.golang.org/

**A Tour of Go日本語版**  
https://go-tour-jp.appspot.com/

# Goのインストール方法
自分のPCでもGoを使いたいときは以下のリンクからインストールする。

https://golang.org/doc/install

# Goの開発環境
各種エディタにプラグインを入れるかJetbrainsからIDEが提供されている。

- Visual Studio Code
  - https://code.visualstudio.com/docs/languages/go
- Atom
  - https://atom.io/packages/go-plus
- Vim
  - https://github.com/fatih/vim-go
Emacs
  - https://github.com/dominikh/go-mode.el
- Goland
  - https://www.jetbrains.com/go/

# (公式以外の)便利なツール
フォーマッタなどは公式ツールとして提供済み。ここではサードパーティのOSSでインストールしておくと便利なものを紹介する。

- delve
  - https://github.com/derekparker/delve
  - GDBライクなデバッガ
- gore
  - https://github.com/motemen/gore
  - irb, iexなどのようなREPL
- richgo
  - https://github.com/kyoh86/richgo
  - go testの結果出力をカラフルに

# 関東のGoの勉強会
関東で行われているGoのコミュニティ。

- golang.tokyo
  - https://golangtokyo.connpass.com/
- Goビギナー
  - https://go-beginners.connpass.com/
- Women Who Go
  - https://womenwhogo-tokyo.connpass.com/
- 横浜Go読書会
  - https://yokohama-go-reading.connpass.com/
- GoCon
  - https://gocon.connpass.com/
- Akiba.go
  - https://akihabarago.connpass.com/
- kamakura.go
  - https://twitter.com/kamakura_golang

# Slack (Global) 
Slack。

- gophers.slack.com
  - 以下のURLから参加できる。
  - https://gophersinvite.herokuapp.com/
- 日本語チャンネルもある。
  - #tokyo
  - #japan

# 参考書籍
日本語のGoの書籍ならば以下がよいと思っている。

- プログラミング言語Go
  - http://amazon.jp/dp/4621300253
- みんなのGo言語【現場で使える実践テクニック】
  - http://amazon.jp/dp/477418392X
- Goならわかるシステムプログラミング
  - http://amazon.jp/dp/4908686033

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=60c659a7abb2c7dda3a8bda5b132808f"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B01LMS7B1O&linkId=ba2078cb70074d6995f4778e3da04293"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4908686033&linkId=ca1623768c07f1a972fcf9c6e0acd040"></iframe>




# 関連
- [当ブログのGo記事一覧](/categories/go/)
- [[書評]Goならわかるシステムプログラミングを読んだ](/2018/02/26/review-go-system-programming)

