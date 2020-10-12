+++
title= "[Go] タグなし switchは switch true {...}と等しい"
date= 2019-11-10T12:50:18+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["golang","gotips"]
keywords = ["Go", "golang", "Go言語", "swtich", "Devquiz"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

先日のgolang.tokyoでは`switch`文に関するDevquizが出題された。
私はわかっていたつもりで乾杯の挨拶中に解説を話したが、間違えた解説だったので改めて仕様を確認した。

<!--more-->

# TL;DR
- 先日のgolang.tokyoのDevquizの解説で誤った解説をしてしまった
    - https://golangtokyo.connpass.com/event/151926/
    - Devquizは`switch ok := false; {...}`という`switch`文の挙動について
- Goの`switch`文は条件以外に、`switch`内にスコープを絞った`Statement`を書くことができる。
    - `switch x := f(); x { ... }`
- Goの`switch`文は条件（タグ）なしで書くことができる
    - タグなし`switch`（`tagless-switch`）は`switch true`

正しい答えは[@makki_d](https://twitter.com/makki_d)さんに会場でも指摘してもらった。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">クイズのswitch文、ok:=falseの後のセミコロンの後が空なのが味噌だよね<a href="https://twitter.com/hashtag/golangtokyo?src=hash&amp;ref_src=twsrc%5Etfw">#golangtokyo</a></p>&mdash; MakKi (@makki_d) <a href="https://twitter.com/makki_d/status/1191667339348963328?ref_src=twsrc%5Etfw">November 5, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# golang.tokyo #27のDevquiz
2019/11/05に行われたgolang.tokyoの抽選では以下のDevquizが出題された。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://golangtokyo.connpass.com/event/151926/" data-iframely-url="//cdn.iframe.ly/Oxh7MyA"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

- https://play.golang.org/p/mSkgdEh8hhw

```go
// 以下のコードを実行すると何が出力されるか答えよ。
package main

import "fmt"

func main() {
    switch ok := false; {
    case true:
        fmt.Println("true", ok)
    case false:
        fmt.Println("false", ok)
    default:
        fmt.Println("default")
    }
}
```

何が出力されるかというと、`true false`が出力される。  
私はこれを「`case true`が常に真として評価されるから`true false`が出力される。」と解説したがそれは誤りだった。

# switch文の言語仕様について
`switch`文の言語仕様は以下で確認することができる。

- Switch statements | The Go Programming Language Specification
    - https://golang.org/ref/spec#Switch_statements

Devquizの`switch`文は`type switch`ではないので、`ExprSwitchStmt`となる。

```
SwitchStmt = ExprSwitchStmt | TypeSwitchStmt .
ExprSwitchStmt = "switch" [ SimpleStmt ";" ] [ Expression ] "{" { ExprCaseClause } "}" .

```
## `switch`内にスコープを絞ったStatementを書ける
`ExprSwitchStmt`の定義を見ればわかる通り、`switch`文は実際に`switch`文で評価したい式（`Expression`）と別に代入などの文（`SimpleStmt`）を含むことができる。

Devquiz中の`switch`文も`SimpleStmt`として、代入式を含んでいた

```go
switch ok := false; {

```

## Goの`switch`文は式（タグ）なしで書くことができる
わかる人はもうわかっただろうが、Devquizの`switch`文には式が含まれていない。これはGoの文法的に間違いではない。

> There can be at most one default case and it may appear anywhere in the "switch" statement. A missing switch expression is equivalent to the boolean value true.
 
 式が含まれていない`switch`文は`switch true`として条件評価が行われる。  
よって、 今回のDevquizは「`switch true`という`switch`文だったので、`case true:`という条件が真になっていた」のが正しい解説となる。  
`switch ok := false; ok {`と書けば`ok`変数の値と各`case`の値が比較され、（人間がパット見考える）期待どおりの挙動となる。

なお、`switch {...}`のような、式がない`switch`文のことを、"タグなし`switch`"（`tagless-switch`）と呼ぶ。  
タグなし`switch`の評価が`switch true`になることはプログラミング言語Goでもしっかり明記されている。


# 終わりに
2,3か月に1回くらいの割合でGoの言語仕様を読んでいる気がする。  
Goの言語仕様は[この1ページ](https://golang.org/ref/spec)だけだし、文中のキーワードにはリンクが付いているので勉強しやすい。  
毎回忘れてやり直している気がするが、`Expression`と`Statement`の違いも頭に入ってきた。


# 参考
- [Switch statements | The Go Programming Language Specification](https://golang.org/ref/spec#Switch_statements)
- [プログラミング言語Go ](http://amazon.jp/dp/4621300253)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=10cf49bd3d559f283cf07fa007e3fae7"></iframe>
