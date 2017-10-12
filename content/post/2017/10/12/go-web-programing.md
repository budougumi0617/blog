+++
title= "[書評]Goプログラミング実践入門　標準ライブラリでゼロからWebアプリを作る"
date= 2017-10-12T08:37:20+09:00
draft = false
slug = ""
categories = ["Go","Web","Review"]
tags = ["golang", "book", "web"]
author = "budougumi0617"
+++

表題の本を読み終わった。

[Goプログラミング実践入門　標準ライブラリでゼロからWebアプリを作る](http://amzn.to/2g2Op1E)

# 前提
自分の`Go`の経験は以下の通り。

- [プログラミング言語Go](http://amzn.to/2ycX1uH)は通読。
- `Go`のプログラミング経験はプログラミング言語Goの練習問題くらい（9割くらいは解いた）
- [`Go`で`REST`叩いてWebページ表示したり`JSON/XML`パースしたり](https://github.com/budougumi0617/gopl/tree/master/ch04/ex14)はやった。
- 上記の本が`1.6`くらいまでの内容なので、`context`とかはまだわかっていない。
- `1.7`以降はみんGoやブログで補完している。


# 感想
Goを使ったRESTの書き方と、必要最低限のデータの取扱方法が載っているので、GoのWebプログラミングの取っ掛かりとしてはよかった。

- httptestを使ったテストの書き方が載っている
- FormつかったPOSTやDBのCRUD/JSON/XMLの扱い方が一通り載ってる
- 1.6ベースなので、`context`周りは別途学習する必要がある
- ピュアGoな内容なので、実践するにはフロントエンドの組み込み方を別途学習する必要がある（これは邦題に期待しすぎたかも）
- リポジトリが公開されているが、紙面のコードだけだと結構断片的
- インプレス社の本なので、結構Kindleで半額になる（2017/10/12現在、半額）

[プログラミング言語Go](http://amzn.to/2ycX1uH)でGoの基礎、この本でGoを使ったWebプログラミングの基礎、[Real World HTTP](http://amzn.to/2yeoPyW)でHTTPとGoの幅広い知識を得る、と渡り歩くと良いかもしれない。

[Goプログラミング実践入門　標準ライブラリでゼロからWebアプリを作る](http://amzn.to/2g2Op1E)