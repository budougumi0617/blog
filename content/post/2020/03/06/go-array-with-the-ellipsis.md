+++
title= "[Go] 省略記号（...）を使った配列宣言の仕方"
date= 2020-03-06T08:40:26+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","gotips", "array", "specification"]
keywords = ["Go", "golang", "array", "配列"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

標準パッケージのコードを眺めていたら、珍しい書き方の部分を見つけた。  
以下のように宣言すると、配列サイズを明示的に書かずに配列を宣言できる。

```go
s := [...]int{1, 2, 3}
```

<!--more-->

# TL;DR
- サイズが固定と分かっているときは配列を使いたい
- `[...]int{1, 2, 3}`と書けば暗黙的にサイズが計算された配列を用意できる。
    - `[...]int{n: 0}`のように書けば`n+1`サイズの配列を用意することも可能
- https://play.golang.org/p/nPFGSATYZwd

```go
s := [...]int{1, 2, 3}
fmt.Printf("%T\n", s) // [3]int

l := [...]int{99: 1}
fmt.Printf("%T\n", l) // [100]int
```

# 配列リテラルの長さに省略記号（`...`）を指定する
標準パッケージのコードを調べていたら、`[...]string{"http:", "https:"}`のような宣言を見つけた。

- https://github.com/golang/go/blob/go1.14/src/cmd/go/internal/get/vcs.go#L641-L648

```go
func httpPrefix(s string) string {
  for _, prefix := range [...]string{"http:", "https:"} {
    if strings.HasPrefix(s, prefix) {
      return prefix
    }
  }
  return ""
}
```

何をしているかというと、固定長サイズ`2`の配列を宣言している。

「[プログラミング言語Go](https://amzn.to/2IpIZMc)」の「`4.1章 配列`」の章で確認すると以下のように明記されていた。

> 配列リテラルで省略記号「`...`」が長さの場所に書かれている場合には、配列長は初期化子の数で決まります。

インデックスによる初期化にも対応しており、次のようにも書ける。

```go
l := [...]int{99: 1}
fmt.Printf("%T\n", l) // [100]int
```

# 終わりに
Goはサイズ違いの配列は比較できない（`[2]int`と`[3]int`は比較できない）。  
配列サイズも変更できないので、盲目的にスライスを使うことが多かった。  
なので、「配列」の使い方は正直あまり覚えていなかった。

冒頭で引用した`range [...]string{"http:", "https:"}`というような`range`の回し方はスマートだと思った。  
また、`SHA256`のような固定長のドメインロジックを使う機会も往々にしてあるので、自分のコードに取り込んでおきたい。

そして、また「[プログラミング言語Go](https://amzn.to/2IpIZMc)」に最初から書いてあった案件だったので、読み直しが必要だ。  
（一度読み込んでいるはずなのに覚えていないことが多すぎる）

# 参考
- [Composite literals | The Go Programming Language Specification](https://golang.org/ref/spec#Composite_literals)
- [プログラミング言語Go](https://amzn.to/2IpIZMc)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=d936cb6b1a4edb520517e88b89bce696"></iframe>
