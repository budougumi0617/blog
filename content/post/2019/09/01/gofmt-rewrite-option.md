+++
title= "[Go] gofmtコマンドの-rオプションの使い方"
date= 2019-09-01T14:15:42+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "gofmt"]
keywords = ["Go", "Golang", "gofmt"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goでは標準ツールとして公式から`gofmt`コマンドというフォーマッタが提供されている。  
このコマンドはコードのインデントなどをフォーマットしてくれるほかに、`-r`オプションでASTベースの置換も行える。  
実装ベースから使い方を追ってみたのでメモする。

<!--more-->

# TL;DR
- `gofmt`コマンドはGoをインストールすると`go`コマンドと一緒にインストールされる公式ツール
- コードのフォーマットを整えてくれる他にASTベースの置換が行える
- `-r`オプションと置換ルールを渡せばよい。
    - `置換前のエクスプレッション -> 置換後のエクスプレッション`
    - 小文字一文字を使うことで置換ルールにワイルドカード指定もできる
- `-s`オプションを使うと、スライス操作などのリファクタリングもすることができる
- 単純にリネームしたいだけなら`gorename`コマンドのほうがよい


なお、執筆時のGoのバージョンは以下の通り。

```bash
$ go version
go version go1.12.9 darwin/amd64
```

# gofmtコマンドのオプションについて
`gofmt`コマンドはGoの公式コードフォーマッタだ。`gofmt`コマンドはGoをソースコードからコンパイルすると`go`コマンドと一緒にビルドされる。  
インデントや改行などを規律に則って実施してくれる。`gofmt`コマンドにはただフォーマットを整える以外に置換する機能も備わっている。

```bash
$ gofmt -h
usage: gofmt [flags] [path ...]
  -cpuprofile string
    	write cpu profile to this file
  -d	display diffs instead of rewriting files
  -e	report all errors (not just the first 10 on different lines)
  -l	list files whose formatting differs from gofmt's
  -r string
    	rewrite rule (e.g., 'a[b:len(a)] -> a[b:]')
  -s	simplify code
  -w	write result to (source) file instead of stdout

```
このオプションの中で`-r`オプションが置換を実行するためのオプションだ。


# -rオプションを使ったコードの置換ルール
上記のコマンドラインのヘルプを見るだけではよくわからないが、`-r`オプションの利用方法の詳細はGoDocに記載されている。

- https://golang.org/cmd/gofmt/

置換ルールは以下のルールに基づいて書く。

- `pattern -> replacement`という形で書く
- `pattern`と`replacement`はGoのエクスプレッションとして正しい文字列を渡す
- エクスプレッション中の小文字一文字はワイルドカードとして解釈される
    - `α[β:len(α)]`だったら`α`と`β`がワイルドカード扱いになる。

実際にコードを確認してみる。`gofmt`コマンドの`-r`オプションの実装は以下にある。

- [https://github.com/golang/go/blob/master/src/cmd/gofmt/rewrite.go](https://github.com/golang/go/blob/d6143914e42e9373fbc02d08b224516b12807269/src/cmd/gofmt/rewrite.go)

`initRewrite`関数をみると、たしかに`->`で文字列を分割し、それぞれがExpressionとして評価できるか確認している。

```go
func initRewrite() {
	if *rewriteRule == "" {
		rewrite = nil // disable any previous rewrite
		return
	}
	f := strings.Split(*rewriteRule, "->")
	if len(f) != 2 {
		fmt.Fprintf(os.Stderr, "rewrite rule must be of the form 'pattern -> replacement'\n")
		os.Exit(2)
	}
	pattern := parseExpr(f[0], "pattern")
	replace := parseExpr(f[1], "replacement")
	rewrite = func(p *ast.File) *ast.File { return rewriteFile(pattern, replace, p) }
}
```

また、`isWildcard`関数で小文字1文字か確認していた。

```go
func isWildcard(s string) bool {
	rune, size := utf8.DecodeRuneInString(s)
	return size == len(s) && unicode.IsLower(rune)
}
```

# 実際の実行例
実際に`gofomt`コマンドでソースコードを置換してみる。置換前のコードは以下になる。

```go
package main

import "fmt"

// Foo is simple function.
func Foo() int {
	return 10
}

// Hoge is sample Struct.
type Hoge struct {
	Foo    string
	foovar int
}

func main() {
	foovar := "same name"
	hoge := Hoge{}
	phoge := &Hoge{}
	hoge.Foo = "Foo"
	phoge.foovar = Foo()

	fmt.Println(hoge.Foo)
	fmt.Println(phoge.foovar)
	fmt.Println(foovar)
}
```

構造体やフィールド名の置換を試みたところ、以下のように編集することができた。

```go
$ gofmt -r "Foo -> Bar" main.go
package main

import "fmt"

// Foo is simple function.
func Bar() int {
	return 10
}

// Hoge is sample Struct.
type Hoge struct {
	Bar    string
	foovar int
}

func main() {
	foovar := "same name"
	hoge := Hoge{}
	phoge := &Hoge{}
	hoge.Bar = "Foo"
	phoge.foovar = Bar()

	fmt.Println(hoge.Bar)
	fmt.Println(phoge.foovar)
	fmt.Println(foovar)
}
```

あくまで構造体やフィールドの置換で、`"Foo"`のような文字列に関しては一緒に置換されないようだ。  
また、構造体コメントも変更はしてくれなかった。

```go
$ gofmt -r "Hoge -> Bar" main.go
package main

import "fmt"

// Foo is simple function.
func Foo() int {
	return 10
}

// Hoge is sample Struct.
type Bar struct {
	Foo    string
	foovar int
}

func main() {
	foovar := "same name"
	hoge := Bar{}
	phoge := &Bar{}
	hoge.Foo = "Foo"
	phoge.foovar = Foo()

	fmt.Println(hoge.Foo)
	fmt.Println(phoge.foovar)
	fmt.Println(foovar)
}
```




# 終わりに
今回は横浜Go読書会で改訂版みんなのGo言語を輪読している際に知った`gofmt`コマンドのオプションについて調べた。  
本当はもっと実装を読み解きたかったのだが、再帰部分（`apply`や`match`関数）をちゃんと理解できなかった。いくつか作りたい静的解析ツールがあるので、時間があるときに読み直して内容を理解したい。

なお、本記事では`-r`オプションしか触れなかったが、`-s`オプションもコードの簡単なリファクタリングをしてくれる。  
実用途してはこちらのほうが扱いやすそうだ。

- https://golang.org/cmd/gofmt/#hdr-The_simplify_command

`-s`オプションは以下のような冗長なコードをリファクタリングしてくれる。

```bash
An array, slice, or map composite literal of the form:
	[]T{T{}, T{}}
will be simplified to:
	[]T{{}, {}}

A slice expression of the form:
	s[a:len(s)]
will be simplified to:
	s[a:]

A range of the form:
	for x, _ = range v {...}
will be simplified to:
	for x = range v {...}

A range of the form:
	for _ = range v {...}
will be simplified to:
	for range v {...}
```

# 参考
- [gofmt を理解する | blog.syfm](https://syfm.hatenablog.com/entry/2018/01/31/011117)
- [Command gofmt | Commands](https://golang.org/cmd/gofmt/)
- [golang のリファクタリングには gofmt ではなく、gorename を使おう。](https://mattn.kaoriya.net/software/lang/go/20150113141338.htm)
- [golang のリファクタリングには gofmt が使える](https://qiita.com/ikawaha/items/73c0a1d975680276b2c7)
