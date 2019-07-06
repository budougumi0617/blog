+++
title= "[Go]スライスを含んだ構造体でruntime errorを回避する #golang"
date= 2019-07-07T09:00:50+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang"]
keywords = ["Go", "Golang"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

<!--more-->


# TL;DR
- Goの値はすべてが`comparable`ではない
    - スライス、`map`、関数の値は`comparable`ではない
- `comparable`ではない値を含んで等価演算子を使うと`runtime error`などが発生する
- 構造体にスライスを含ませながら等価演算子を使いたいならばスライスのポインタで内包すれば良い
    - ただし当然ポインタの指し示す値の比較で等価であるか決まるようになる

# Goの値はすべてがcomparableではない
プログラミング言語によってはすべてのオブジェクトが`equality`を持ち、`comparable`であることもある。
が、Goの場合そうではない。Goは仕様に`compareble`でない型があることを明記している。

- Comparison operators
    - https://golang.org/ref/spec#Comparison_operators

> Slice, map, and function values are not comparable. However, as a special case, a slice, map, or function value may be compared to the predeclared identifier nil. Comparison of pointer, channel, and interface values to nil is also allowed and follows from the general rules above.


上記の通り、インターフェースに`not comparable`な値が入っている状態で比較を行うと`runtime error`が発生する。
`not comparable`な値とはスライスや`map`、`function`への値のことを指す。


# `comparable`ではない値を含んで等価演算子を使うと`runtime error`などが発生する
この`comparable`性については構造体にも影響する。`not comparable`な型のフィールドを持つ構造体も`not comparable`になる。
そして、`not comparable`な値を等価演算子で比較すると、`runtime error`が発生する旨も仕様に記載されている（記載場所は上記リンクと同じ）。

> A comparison of two interface values with identical dynamic types causes a run-time panic if values of that type are not comparable. This behavior applies not only to direct interface value comparisons but also when comparing arrays of interface values or structs with interface-valued fields.

下記のサンプルコードは`[]string`を内包する構造体を比較してしまうため、`runtime error`が発生する。

```go
// https://play.golang.org/p/FO_vwjsT7TY
package main

import (
	"fmt"
)

type Article struct {
	Title string
	Tag   []string
}

func main() {
	var a1 interface{} = Article{
		Title: "Content Title",
		Tag: []string{
			"go",
			"slice",
		},
	}

	var a2 interface{} = Article{
		Title: "Content Title",
		Tag: []string{
			"go",
			"slice",
		},
	}
	// panic: runtime error: comparing uncomparable type main.Article
	if a1 == a2 {
		fmt.Println("Same article")
	}
}
```

# 構造体にスライスを含ませながら等価演算子を使いたいならばスライスのポインタで内包すれば良い
このような構造体を宣言してしまったとき、利用者からすると`runtime error`が発生するような構造体を返ってくるのは不便かもしれない。


```go
// https://play.golang.org/p/6RGzUoPFrjW
package main

import (
	"fmt"
)

type Tags *[]string

type Article struct {
	Title string
	Tag   Tags
}

func main() {
	var a1 interface{} = Article{
		Title: "Content Title",
		Tag: &[]string{
			"go",
			"slice",
		},
	}

	var a2 interface{} = Article{
		Title: "Content Title",
		Tag: &[]string{
			"go",
			"slice",
		},
	}
	if a1 == a2 {
		fmt.Println("Same article")
	}
}
```

# 終わりに
業務で実際に`runtime error`が発生していたのでどうしたら回避できるのか調べたのが今回の記事になる。
「そういえば`pkg/errors`はスライスで`StackFrame`持っているはずだけど同じエラーが発生しないな？」と調べたらポインタを使っていた。
著名なOSSのコードはやはり勉強になるので使うだけでなくちゃんと読んでみないとなと改めて思った。

```go
// https://github.com/pkg/errors/blob/27936f6d90f9c8e1145f11ed52ffffbfdb9e0af7/errors.go#L119-L123
// fundamental is an error that has a message and a stack, but no caller.
type fundamental struct {
	msg string
	*stack // type stack []uintptr
}
```
