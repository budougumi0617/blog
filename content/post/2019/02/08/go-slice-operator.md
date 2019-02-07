+++
title= "[Go]スライス生成時(slice operator)に使える珍しい宣言方法"
date= 2019-02-08T07:24:25+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang"]
keywords = ["Go", "Golang", "slice", "初心者", "スライス"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goにおけるスライス演算子(*slice operator*)を改めて調べ直し、3-INDEX記法などを学んだ。

<!--more-->

# TL;DR
- スライスは`make`などのほかにスライス演算子(*slice operator*)による初期化も存在する
- スライス演算子でもcapacityを指定することができる。
- 他にも珍しいスライスの宣言方法があった。

```go
arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

s := arr[2:5]           // s = [2 3 4] cap(s) = 8
s = arr[2:5:7]          // s = [2 3 4] cap(s) = 5
s = arr[:0]             // s = []      cap(s) = 10
s = arr[:0:7]           // s = []      cap(s) = 7
s = make([]int, 10)[:5] // [0 0 0 0 0]  make ([]int, 5, 10)と同じ
```

記事中のサンプルコードをまとめたplaygroundは以下。

- https://play.golang.org/p/rS6SU9m5v2F

# スライス
Goにおけるスライスは、基底配列(*underlying array*)の要素の部分列（あるいは全部）へアクセスする軽量なデータ構造だ。

- Slice expressions | The Go Programming Language Specification
  - https://golang.org/ref/spec#Slice_expressions

詳細なデータ構造についてはGo Blogが参考になる。

- Go Slices: usage and internals | The Go Blog
  - https://blog.golang.org/go-slices-usage-and-internals

スライスは大雑把に以下のような情報を持つ。容量を超える数の値が入る場合はメモリアロケーションが走る。

- 基底配列の特定の場所を指すポインタ。スライスの先頭要素になる場所
- 長さ。スライスにふくまれている値の数。`len`関数で取得できる値
- 容量(*capacity*）。`cap`関数で取得できる値

まったく新しいスライスを宣言するときは`make`などを使う。

```go
make([]int, 5)       // [0 0 0 0 0]
make([]int, 5, 10)   // [0 0 0 0 0] あらかじめ容量が10ある
[]int{}              // [0 0 0 0 0]
[]int{3:2, 5:1}      // [0 0 0 2 0 1]
```

詳細はプログラミング言語Go 4.2スライスに記載されている（上記も大半が引用）。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=d133cb0683bd924a3f87eb96c455af48"></iframe>

# スライス演算子(*slice operator)*

そしてある配列（あるいは別のスライス）から新たなスライスを生成するのがスライス演算子(*slice operator*)だ。
基底配列（あるいは別のスライス）のオブジェクト`array`に対して`array[1:2]`のように宣言する。

例えば以下のようにスライス演算子でスライスを生成すると、各情報は以下のようになる。
```
	arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9} // array(not slice)
	s := arr[2:5]                                 // slice operator a[ low : high ]

	fmt.Println("s =", s)        // s = [2 3 4]
	fmt.Println("len =", len(s)) // len = 3
	fmt.Println("cap =", cap(s)) // cap = 8
```

上記の例でいうと、「基底配列の2番目の要素」(`law`)から「基底配列の(5-1)番目の要素」(`high`)までのスライスが用意される。
容量は基底配列（あるいは元のスライス）で決まるので、`基底配列の長さ - スライスの開始位置(len(arr) - low))`になる。

# スライス演算子の省略記法
スライス演算子では`low`や`high`を明示的に書かずに省略することができる。

```go
a[2:]  // a[2 : len(a)] と書くのと同じ
a[:3]  // a[0 : 3] と書くのと同じ
a[:]   // a[0 : len(a)] と書くのと同じ
```

# 3-INDEX記法でスライス演算子利用時でも容量を指定する
あまり見ない気がするが、3つインデックスを指定する方法も存在する。

- Three-index slices | Go 1.2 Release Notes
  - https://golang.org/doc/go1.2#three_index

`make`を使ってスライスを生成するとき(`make([]int, len, cap)`)のように、容量を3つ目の値として指定する。


```
arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9} 

s = arr[2:5:7]
fmt.Println("s =", s)        // s = [2 3 4]
fmt.Println("len =", len(s)) // len = 3
fmt.Println("cap =", cap(s)) // cap = 5
```

3-INDEX記法でも最初のパラメータだけを省略することができる。2番目と3番目のパラメータは省略することができなくなる。

```go
arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

s = arr[:0:7]
fmt.Println("s =", s)        // s = []
fmt.Println("len =", len(s)) // len = 0
fmt.Println("cap =", cap(s)) // cap = 7

// s = arr[::7]              // middle index required in 3-index slice
// s = arr[:5:]              // final index required in 3-index slice
// s = arr[5:7:5]            // invalid slice index: 7 > 5
```


# 最後に
スライスの宣言について仕様を再確認していたら結構知らなかったことがあったので、まとめた。
3-INDEX記法はいつから入ったんだ？と思ったが1.2からだったのでほぼ最初から入っていたのに使ったことがなかった。
普段メモリアロケーションなど細部を気をつけてコーディングできていないので状況に応じては使えるようになった（そんな状況がいつあるか？はちょっと想像できていないが…）。
基底配列(*underlying array*)や`a[low:high]`という書き方を *slice operator* と呼ぶこともわかってよかった。ただ、今回調べていて一番びっくりした書き方は以下だったりする。


```
// [0 0 0 0 0] make ([]int, 5, 10)と同じ
make([]int, 10)[:5]
```

# 参考
- Slice expressions | The Go Programming Language Specification
  - https://golang.org/ref/spec#Slice_expressions
- Three-index slices | Go 1.2 Release Notes
  - https://golang.org/doc/go1.2#three_index
- Go Slices: usage and internals | The Go Blog
  - https://blog.golang.org/go-slices-usage-and-internals

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=d133cb0683bd924a3f87eb96c455af48"></iframe>
