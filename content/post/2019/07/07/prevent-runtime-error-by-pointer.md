+++
title= "[Go]Sliceを含んだ構造体が等値演算子（==）でpanicを引き起こすのを回避する #golang"
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


Goには`comparable`が定義されておらず、比較できない型として`Slice`, `Map`などがある。  
`interface`がそのような型（フィールドにそのような型を持った構造体）を値に持っていたときに`==`を利用すると`panic`が発生する可能性がある。  
行儀の悪い構造体を定義しないテクニックがあったのでメモしておく。

<!--more-->

# TL;DR
- Goの値はすべてが`comparable`ではない
    - Slice、`Map`、関数の値は`comparable`ではない
- `comparable`ではない値を指すインターフェースに等価演算子を使うと`runtime error`が発生する
- 構造体にSliceを含ませながら等価演算子を使いたいならばSliceのポインタで内包すれば良い
    - ただし当然ポインタの指し示す値の比較で等価であるか決まるようになる

# Goの値はすべてがcomparableではない
プログラミング言語によってはすべてのオブジェクトが`equality`を持ち、`comparable`であることもある。
が、Goの場合そうではない。Goは仕様に`compareble`でない型があることを明記している。

- Comparison operators
    - https://golang.org/ref/spec#Comparison_operators

> Slice, map, and function values are not comparable. However, as a special case, a slice, map, or function value may be compared to the predeclared identifier nil. Comparison of pointer, channel, and interface values to nil is also allowed and follows from the general rules above.


上記の通り、`not comparable`な値とはSliceや`Map`、`function`への値のことを指す。


# `comparable`ではない値を指すインターフェースに等価演算子を使うと`runtime error`が発生する
この`comparable`性については構造体にも影響する。
構造体が`not comparable`な型のフィールドを持つとき、その構造体も`not comparable`になる。  
明示的に`not comparable`な構造体を比較しているときはビルド時にエラーになるので気づく。


```go
 a1 := struct {
 	Slice []string
 }{
 	[]string{
 		"hoge",
 	},
 }
 
 // invalid operation: a1 == a1 (struct containing []string cannot be compared)
 if a1 == a1 {
 	fmt.Println("Same article")
 }
```


問題は`interface`でそのような構造体を指していたときだ。  
`not comparable`な値を指した状態の`interface`値を等価演算子で比較すると、`runtime error`が発生する旨が仕様に記載されている（記載場所は上記リンクと同じ）。

> A comparison of two interface values with identical dynamic types causes a run-time panic if values of that type are not comparable. This behavior applies not only to direct interface value comparisons but also when comparing arrays of interface values or structs with interface-valued fields.

下記のサンプルコードは`interface`を介して`[]string`を内包する構造体を比較してしまう。
そのため、ビルドには成功するが、実行時に`runtime error`が発生する。

- https://play.golang.org/p/FO_vwjsT7TY

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

サンプルコードでは`not comparable`な値が入っているのは自明だが、関数やメソッドの引数を`interface`としていたとき、渡ってきた具象型が`comparable`かどうかわざわざ検証しないだろう。
`error`インターフェースなど、具象型を意識しないでで取り扱う構造体は`comparable`な構造体にしておきたい。

# 構造体をcomparableにしてruntime errorを回避するにはポインタを使ってフィールドを定義する

では具体的にどのように回避すればよいかというと、ポインタを利用することでSliceを内部で持っていたまま`comparable`な構造体にできる（Sliceの場合は配列にしてもよい）。
サンプルコードは以下。

- https://play.golang.org/p/6RGzUoPFrjW

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

`[]string`ではなく、`*[]string`にした構造体を宣言すると`runtime error`は解消された。  
ただ、当然ポインタで比較されるようになるのでそこは注意する。
なので、サンプルコードの`a1`と`a2`は直感的には「等価」だが、ポインタの指すSliceは別物なので「等値」ではなく`!=`になる。
あくまで何も知らないで`interface`を経由して利用される場合の`runtime error`回避のためのテクニックなので、構造体の中身の詳細を知っているコードでは`reflect.DeepEqual`などを利用する必要がある。

# 終わりに
業務で実際に`runtime error`が発生する可能性があったので、どうしたら回避できるのか調べたのが今回の記事になる。  
「そういえば`pkg/errors`はSliceで`StackFrame`持っているはずだけど同じエラーが発生しないな？」と調べたらポインタを使っていた。  
著名なOSSのコードはやはり勉強になるので使うだけでなくちゃんと読んでみないとなと改めて思った。

```go
// https://github.com/pkg/errors/blob/27936f6d90f9c8e1145f11ed52ffffbfdb9e0af7/errors.go#L119-L123
// fundamental is an error that has a message and a stack, but no caller.
type fundamental struct {
	msg string
	*stack // type stack []uintptr
}
```

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/pkg/errors" data-iframely-url="//cdn.iframe.ly/awcJrVL"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 追記
比較する2つのオブジェクトが違う型ならば`runtime error`が発生するnot comparableなフィールドを触る前に比較が終わるので`runtime error`は発生しない。
なので、比較可能な具象型をどちらかに置いて比較演算子を書けば`runtime error`が出ないことは保証される（`interface == interface`と比較するケースはあまりないだろう）。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">実装まで見れてませんが、型が違えば型比較でEqualityの判定が終わるので平気なんですよね。なので比較するどちらかがcomparebleならruntime errorが出ないことは保証できます。<br>ただどう使われるかわからないライブラリ公開者は気をつけて実装したほうがいいという話です。<a href="https://t.co/5x924VN9aK">https://t.co/5x924VN9aK</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1148715295772557313?ref_src=twsrc%5Etfw">July 9, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

```go
package main

import (
	"fmt"
)

type NotCompareble []string

func main() {
	var a interface{} = []string{
		"go",
		"slice",
	}

	var b interface{} = NotCompareble{
		"go",
		"slice",
	}
	// not runtime error, even if each type are not comparable, because a, b are different type.
	if a != b {
		fmt.Println("different object")
	}
}
```

