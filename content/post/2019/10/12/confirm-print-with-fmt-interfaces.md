+++
title= "[Go] 独自型にfmtパッケージのインターフェースを実装して出力を制御する"
date= 2019-10-12T15:17:06+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["golang","fmt","gotips", "Stringer", "GoStringer"]
keywords = [""]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

`fmt`パッケージには`fmt.Printf`の出力を任意に変更できるインターフェースが定義されている。  
各インターフェースを満たす独自型をフィールドに持つ構造体の出力がどうなるのか確認し、任意の型の出力を制御できるか確認してみた。

<!--more-->

# TL;DR
- `fmt.Printf`関数はPrint verbによって出力形式を変更できる
  - https://golang.org/pkg/fmt/#hdr-Printing
- `fmt`パッケージのインターフェースを実装すると出力を制御できる
  - `fmt.Stringer`インターフェース
      - https://golang.org/pkg/fmt/#Stringer
  - `fmt.GoStringer`インターフェース
      - https://golang.org/pkg/fmt/#GoStringer
  - `fmt.Formatter`インターフェース
      - https://golang.org/pkg/fmt/#Formatter
- インターフェースを実装した独自型は構造体に含まれても出力が制御できる
- これを使えば独自型の出力に任意のマスキング処理や追加情報を付与できそう
  - https://play.golang.org/p/zHvg22dxkX_N

# StringメソッドやGoStringメソッドを実装してPrint出力の結果を整形する。

Goは他の言語と同じように`fmt.Printf`関数でPrint verbを使うことで出力される情報量を制御できる。

- Printing | fmt package
  - https://golang.org/pkg/fmt/#hdr-Printing

例えば、`string`型は`%s`で表示し、構造体は`%v`を使う。`%+v`を使うと構造体フィールド名も一緒に出力される。
これらの出力は`fmt`パッケージに定義されているインターフェースを実装することで、任意の出力形式に変更できる。

- https://play.golang.org/p/EWOEsNcmd3Z

```go
package main

import (
	"fmt"
	"testing"
)

type MyString string

func (ms MyString) String() string {
	return "return from String()"
}

func (ms MyString) GoString() string {
	return "return from GoString()"
}

func TestMyString(t *testing.T) {
	var ms MyString
	fmt.Println(ms)
	fmt.Printf("ms by %%s\t=\t%s\n", ms)
	fmt.Printf("ms by %%v\t=\t%+v\n", ms)
	fmt.Printf("ms by %%+v\t=\t%v\n", ms)
	fmt.Printf("ms by %%#v\t=\t%#v\n", ms)
}
```

例えば、上のコードの`MyString`型は`fmt.Stringer`インターフェースと、`fmt.GoStringer`インターフェースを実装している。

- `fmt.Stringer`インターフェース
  - https://golang.org/pkg/fmt/#Stringer
- `fmt.GoStringer`インターフェース
  - https://golang.org/pkg/fmt/#GoStringer

そのため、`fmt.Printf`関数で対応した`Print verb`を使って出力すると、以下のような出力結果となる。
`String`メソッドや`GoString`メソッドの実装結果が出力されているのがわかる。


```bash
$ go test -v fmt_test.go
=== RUN   TestMyString
return from String()
ms by %s	=	return from String()
ms by %v	=	return from String()
ms by %+v	=	return from String()
ms by %#v	=	return from GoString()
--- PASS: TestMyString (0.00s)
PASS
```

`fmt.Formatter`インターフェースを実装すれば`%s`や`%v`以外の`verb`の出力結果を変更できる。

- `fmt.Formatter`インターフェース
  - https://golang.org/pkg/fmt/#Formatter

具体的な詳細は@tenntennさんのQiitaの記事を見ればよいだろう。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://qiita.com/tenntenn/items/453a09c4c6d7f580d0ab" data-iframely-url="//cdn.iframe.ly/m4WA0zK?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# Stringerインターフェースなどを実装した型をフィールドに持つ構造体の出力
では、構造体のフィールドにこのように出力形式を変更している型を指定するとどのように出力されるのだろう？
結論から言うと、型に実装した`Stringer`インターフェースの内容に沿った出力がされた。

- https://play.golang.org/p/GRuqRLY8JZ9

```go
package main

import (
	"fmt"
	"testing"
)

type MyString string

func (ms MyString) String() string {
	return "return from String()"
}

func (ms MyString) GoString() string {
	return "return from GoString()"
}

type Root struct {
	RootField MyString
}

func TestRoot(t *testing.T) {
	root := Root{}
	fmt.Println(root)
	fmt.Printf("root = %+v\n", root)
	fmt.Printf("root by %%s\t=\t%s\n", root)
	fmt.Printf("root by %%v\t=\t%v\n", root)
	fmt.Printf("root by %%+v\t=\t%+v\n", root)
	fmt.Printf("root by %%#v\t=\t%#v\n", root)
}
```

上記の`Root`構造体はフィールドに`MyString`型を持つ。`Root`オブジェクトの出力は`MyString`型に実装された`String`メソッドなどを反映した内容になっている。

```bash
$ go test -v fmt_test.go
=== RUN   TestRoot
{return from String()}
root = {RootField:return from String()}
root by %s	=	{return from String()}
root by %v	=	{return from String()}
root by %+v	=	{RootField:return from String()}
root by %#v	=	main.Root{RootField:return from GoString()}
--- PASS: TestRoot (0.00s)
PASS
```

# マスキングなどに使えそう。
ではこの機能をどう使うのか？間接的にも実装したインタフェースの結果が反映されるので、マスキングなどに使えそうだ。
例えば、アプリケーションを作成するとき、パスワードなどの機密情報はそのままログ出力されてほしくない。


- https://play.golang.org/p/oDmOKOim5xl

```go
package main

import (
	"fmt"
	"testing"
)

type Password string

func (p Password) String() string {
	rs := []rune(p)
	for i := 0; i < len(rs)-2; i++ {
		rs[i] = 'X'
	}
	return string(rs)
}

type Credential struct {
	ID       string
	Password Password
}

func TestPassword(t *testing.T) {
	cr := Credential{
		ID:       "budougumi0617",
		Password: "secret",
	}
	fmt.Println(cr)
	fmt.Printf("cr by %%s\t=\t%s\n", cr)
	fmt.Printf("cr by %%v\t=\t%v\n", cr)
	fmt.Printf("cr by %%+v\t=\t%+v\n", cr)
	fmt.Printf("cr by %%#v\t=\t%#v\n", cr)
}
```

`GoStringer`インターフェースを実装していないので、`%#v` verbを使った出力はマスキングできていないが、他の出力についてはマスキングができている。  
ロガーなどでマスキングを仕込むより、`Password`型などの独自型でマスキングを定義したほうが漏れがなくマスキングを行えそうだ。

```bash
$ go test -v fmt_test.go
=== RUN   TestPassword
{budougumi0617 XXXXet}
cr by %s	=	{budougumi0617 XXXXet}
cr by %v	=	{budougumi0617 XXXXet}
cr by %+v	=	{ID:budougumi0617 Password:XXXXet}
cr by %#v	=	main.Credential{ID:"budougumi0617", Password:"secret"}
--- PASS: TestPassword (0.00s)
PASS
```

# 参考
- fmt.Formatterを実装して%vや%+vをカスタマイズしたり、%3🍺みたいな書式をつくってみよう #golang
  - https://qiita.com/tenntenn/items/453a09c4c6d7f580d0ab
- `fmt.Stringer`インターフェース
  - https://golang.org/pkg/fmt/#Stringer
- `fmt.GoStringer`インターフェース
   - https://golang.org/pkg/fmt/#GoStringer
- `fmt.Formatter`インターフェース
   - https://golang.org/pkg/fmt/#Formatter

