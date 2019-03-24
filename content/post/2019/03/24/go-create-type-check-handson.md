+++
title= "go/typesパッケージを使って型チェックを行なうハンズオンを作った #golangtokyo"
date= 2019-03-24T19:04:50+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["go", "static analysis", "types"]
keywords = ["Go", "静的解析", "type check", "go/types", "golang"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

golang.tokyoで静的解析ハンズオンを行なうにあたり、ハンズオン資料を作成した。  
ハンズオン作成時の際参考にした資料や、作る時に工夫した点をまとめておく。

- 型チェックでインターフェースを実装しているか調べよう
    - https://golangtokyo.github.io/codelab/first-step-type-check/
<!--more-->

# TL;DR
- `go/types`パッケージを使った型チェックを勉強するハンズオンを作った
    - [型チェックでインターフェースを実装しているか調べよう](https://golangtokyo.github.io/codelab/first-step-type-check)
- ハンズオンを作るにあたって以下のWEBサイト・記事が大変参考になった
- 資料を作る際はひたすらPlayGroundで以下のようなサンプルコードを回していた
    - https://play.golang.org/p/DltIoOjroh2
- 静的解析を勉強して「Goにおける型」の概念を知ることができた。言語仕様に対する理解も深めることができてオススメ。

なお、静的解析ハンズオン自体の情報は以下になる。

|イベント名|golang.tokyo #22+Okayama.go/Sendai.go|
|---|---|
|URL|https://golangtokyo.connpass.com/event/122263/|
|会場|freee株式会社 東京都品川区西五反田2-8-1(五反田ファーストビル 9F)|
|日時|2019/03/23（土）13:00 〜 17:00|
|ハッシュタグ|[#golangtokyo](https://twitter.com/hashtag/golangtokyo)|

# なぜ作ったのか
今回golang.tokyoで静的解析のハンズオンをすることになった。  
golang.tokyoでは以前から静的解析を含めたハンズオンを提供していたが型チェック（`go/types`）についてのハンズオンはまだなかった。  
私自身今年のはじめあたりから静的解析を初めたが、まだ`go/types`パッケージはあまり触ったことがなかったのでこの機会に学習することにした。

# 参考にした記事
今回の資料を作るにあたって以下の記事を参考にした。また、そもそもとして、ハンズオン資料のネタ出しは[@tenntenn](https://twitter.com/tenntenn)さんにしてもらった。  
（全てのネタをハンズオンに含めることはできなかったのだが、）自分にとっていい具合にストレッチした課題（ネタ）だった。圧倒的に感謝！

## go/typesのGodoc
- https://golang.org/pkg/go/types/

当たり前だが`go/types`のGo Doc。`types.Info`構造体などを使うときは一度Go Docを読んでExamplesを動かしておいたほうが圧倒的に理解が進むと思う。

- https://golang.org/pkg/go/types/#Info

## GoのためのGo
- https://motemen.github.io/go-for-go-book

[@motemen](https://twitter.com/motemen)さんが書いている静的解析全般の解説記事。日本語で`go/ast`パッケージや`go/types`パッケージの内容がまとめてあって大変ありがたい。  
また、資料内にある[Appendix A: ast.Nodeの階層](https://motemen.github.io/go-for-go-book/#ast_node%E3%81%AE%E9%9A%8E%E5%B1%A4)などが大変参考になった。  
Go Docはたどりやすいし実際のコードもワンクリックで辿れるのでとても読みやすいのだが、インターフェースと具象型の関係などを調べることができない（ダックタイピングなので当然といえば当然）。  
なので、「この型を取りたいときはどのインターフェースから辿れたいいのいかな？」などを調べるときにとても重宝した。

- [Appendix A: ast.Nodeの階層](https://motemen.github.io/go-for-go-book/#ast_node%E3%81%AE%E9%9A%8E%E5%B1%A4)
- [Appendix B: types.Objectの階層](https://motemen.github.io/go-for-go-book/#types_object%E3%81%AE%E9%9A%8E%E5%B1%A4)
- [Appendix C: types.Typeの階層](https://motemen.github.io/go-for-go-book/#types_type%E3%81%AE%E9%9A%8E%E5%B1%A4)

## go/types: The Go Type Checker
- https://github.com/golang/example/tree/master/gotypes

`godoc -analysis`などGoの静的解析部分の設計者であり、[プログラミング言語Go](http://amazon.jp/dp/4621300253)執筆者のAlan Donovan氏による`go/types`パッケージの解説。
英語が読めないのでちゃんと全部は読めていないのだが、サンプルコードも大量にありわかりやすい。

## 静的解析で型を扱う #golang
- https://qiita.com/tenntenn/items/dfc112ae7bb8c9703ab9

[@tenntenn](https://twitter.com/tenntenn)さんが書いた型の記事。
Goには継承はないが、静的解析的（構文木）には「基底型」という概念がある。`error`型の基底型を取り出す方法、基底型の概念を学べた。  
インターフェースを実装しているか？などの比較処理なども書いてあり、ハンズオン中のコードはだいぶこの記事から作られている。  
今回ハンズオン資料を作るにあたって一番の自分にとっての収穫は基底型という概念とGoの中の型の概念を知れたところが大きい。

## goでstructの定義を見てあれこれするのに、ast.Fileを直接見るよりtypes.Packageを参照したほうが楽(かもしれない)
- https://pod.hatenablog.com/entry/2018/02/04/145615

[@podhmo](https://twitter.com/podhmo)さんが書いた`types.Object`の概観の記事。  
最初は`types.Object`と`types.Type`の違いもよくわかっていなかったので全体感を感じられてよかった。  
あと、記事の最後に添付されている[`gist`](https://gist.github.com/podhmo/85aec6afb53df58d35f4264fa756b952)のサンプルコードがいい感じの分量のコードで理解が捗った。  
なんだかんだ静的解析をするときはひたすら動かしてキャストを繰り返したり挙動を確認するしかないので、このサンプルコードをベースにPlaygroudをいろいろ動かしてみて理解を進めた。

# 作る時に工夫したこと
私は動かさないとわからないタイプだったのでひたすらPlaygroudでサンプルコードを動かして理解していた。  
具体的には下記のようなコードをちょっとずつ変えながらひたすらデバッグプリントをしたりしていた。

```go
package main

import (
	"fmt"
	"go/ast"
	"go/importer"
	"go/parser"
	"go/token"
	"go/types"
	"log"
)

const code = `
package p

type I interface{
	Hoge() string
}

type S struct {
}

func (s *S) Hoge() string{
	return ""
}

type Y struct {
}

func (y Y) Error() string{
	return ""
}

`

func main() {
	fset := token.NewFileSet()

	conf := types.Config{
		Importer: importer.Default(),
		Error: func(err error) {
			fmt.Printf("!!! %#v\n", err)
		},
	}

	f, err := parser.ParseFile(fset, "p", code, parser.ParseComments)
	if err != nil {
		log.Fatal(err)
	}

	// Info.ObjectOfやInfo.TypeOfのspecに書いてあるPreconditionを読んで初期化が必要なフィールドを把握しておく
	info := &types.Info{
		// Types: map[ast.Expr]types.TypeAndValue{},
		Defs: map[*ast.Ident]types.Object{},
		// Uses:  map[*ast.Ident]types.Object{},
	}

	_, err = conf.Check("p", fset, []*ast.File{f}, info)
	if err != nil {
		log.Fatal(err)
	}

	// error型の定義を取得する
	errType := types.Universe.Lookup("error").Type()
	// error型の基底となっているインターフェースを取得する
	it := errType.Underlying().(*types.Interface)

	ast.Inspect(f, func(n ast.Node) bool {
		switch n := n.(type) {
		// 識別子のみ処理する
		case *ast.Ident:
			// types.Infoの中からtypes.Objectを取得する
			obj := info.ObjectOf(n)
			if obj == nil {
				return true
			}
			// 型名じゃなかったら無視する
			if _, ok := obj.(*types.TypeName); !ok {
				return true
			}
			typ := obj.Type()


			// 構造体か
			if _, ok := typ.Underlying().(*types.Struct); ok {
				if types.Implements(typ, it) {
					fmt.Println(fset.Position(obj.Pos()), typ, "implements", errType)
					return true
				}
				// ポインタ型としてerrorインターフェースを満たしていないか
				ptr := types.NewPointer(typ)
				if types.Implements(ptr, it) {
					fmt.Println(fset.Position(obj.Pos()), ptr, "implements", errType)
				}
			}
		}
		return true
	})
}
```

今は[`x/tools/go/analysis`](https://godoc.org/golang.org/x/tools/go/analysis)パッケージもあるので、`analysistest`パッケージを使ってひたすらテストを実行しても良いかも知れない。

# 終わりに
- 型チェックでインターフェースを実装しているか調べよう
    - https://golangtokyo.github.io/codelab/first-step-type-check/

今回はgolang.tokyoに寄稿したハンズオン資料作成時のいろいろを自分用にまとめた。今後は実際に静的解析ツールを作るのと、もっと静的解析について勉強をしていきたい。  
本当はanalysisパッケージを使ってCLIツールを作るまでやりたかったのだが、自分の力量不足でできなかったので、そのようなハンズオン資料もいずれ作っていきたい。  
静的解析を勉強すると言語仕様についての理解も進んでいくことがわかったので以前以上にやる気も出てきた。  
また、ハンズオン資料の作成で使っているCodlabsのツールはとても使い勝手がいいので、ツールの使い方を紹介する記事を後日書いておく。


ちなみにハンズオン資料の不備などは以下で受け付けているので、間違いやわかりにくいところがあったらぜひ登録していただけると幸い。

- https://github.com/golangtokyo/codelab/issues

# 参考URL
- go/typesパッケージ
    - https://golang.org/pkg/go/types
- go/types: The Go Type Checker
    - https://github.com/golang/example/tree/master/gotypes
- GoのためのGo
    - https://motemen.github.io/go-for-go-book/
- 静的解析で型を扱う #golang
    - https://qiita.com/tenntenn/items/dfc112ae7bb8c9703ab9
- goでstructの定義を見てあれこれするのに、ast.Fileを直接見るよりtypes.Packageを参照したほうが楽(かもしれない)
    - https://pod.hatenablog.com/entry/2018/02/04/145615
