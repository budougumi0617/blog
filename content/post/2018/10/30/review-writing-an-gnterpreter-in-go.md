+++
title = "[書評] Go言語でつくるインタプリタ を読んだ #go"
date = 2018-10-30T10:48:30+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["review", "go"]
tags = ["golang", "book"]
+++



[Go言語でつくるインタプリタ](https://amazon.jp/dp/4873118220)を読んだのでメモ。
プログラマなら一度はやる（？）であろうインタプリタの実装をGoで行う本書では、優れた設計は拡張性が高く「小さく作る」・「常に動くものを作る」を可能にすること、テスト駆動開発とテーブル駆動テストを用いることで検証を繰り返しながら着実に開発を進めることができることを実感することができる。

<!--more-->

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873118220&linkId=ce2985386081de40a6b08a555d61ebe0"></iframe>

# なぜ読んだのか？
一言で言うとGoの勉強とパーサーを書きたいと思っていたので手にとった。以前の職場の知り合いに聞くたびに何らかの理由(あるフレームワークのlinterを作りたいetc）でパーサーを書いているめちゃくちゃできる人がいて、「パーサーを書く」という行為に憧れのような感情を抱いていた。「パーサーを作る」という行為は以下のスキルがないと実現できないからだ。

- パース対象の式表現を抽象構文木（abstract syntax tree、AST）に落とし込む
- ASTを操作・走査する実装力

正直Go言語の場合はAST pkgがあるので「それ見ろよ」でもいいのかもしれない。ただ最初から相当規模となっている完成形を見るのではなく、ゼロから作り始めてひとつひとつを解説してくれるほうがよかった。本書は英語版を[@deeeet](https://twitter.com/deeeet)さんが紹介していたりして気になっていた。

- Package ast
  - https://golang.org/pkg/go/ast/
- Writing An Interpreter In Goを読んだ
  - https://deeeet.com/writing/2017/01/12/go-interpreter/

# 実装するインタプリタとMonkyという仮想言語について
本書ではMonkeyという仮想言語のインタプリタの開発を行う。最終的に実装するインタプリタではMonkey言語は以下のような機能が実現される（本書の「はじめに」より引用）。

- C言語風の構文
- 変数束縛
- 整数と真偽値
- 算術式
- 組み込み関数
- 第一級の高階関数(関数を引数に受け取る関数)
- クロージャ
- 文字列データ型
- 配列データ型
- ハッシュデータ型

また、本書は以下のWebサイトから購入できる`Writing An Interpreter In Go`の翻訳版だ。

- Writing An Interpreter In Go
  - https://interpreterbook.com/

翻訳版の本書は無料公開されている以下のおまけチャプターの「マクロシステムの実装」も付録として翻訳されている。

- The Lost Chapter - A Macro System For Monkey
  - https://interpreterbook.com/lost/

よって、上記の箇条書きの機能に加えElixerのquote/unquoteのようなマクロ機能も実装する。

- Quote and unquote
  - https://elixir-lang.org/getting-started/meta/quote-and-unquote.html

本書ではこれらの機能の実装（インタプリタの実装）をゼロから（字句解析、構文解析から）始めて全て行う。
後述する通り、本書では第一章からREPLとしても作成するので、実際にコンソールで機能を確かめながら実装を進めていく。

# 「プログラムを作る」過程に必要なアプローチを学ぶことができる。

この本の良さに以下の2つがある。なので、「インタプリタを作る」ことの勉強だけではなく、「何もないところからプログラムを作る」過程のアプローチについても学ぶことができる。

- 終始「動くものを小さく作る」を一貫して最後まで続ける
- テスト駆動開発（TDD）で機能を着実に実装していく

## 終始「動くものを小さく作る」を一貫して最後まで続ける
本書のインタプリタではASTのノードを全て`Node`インターフェースで表現する。与えられたテキストは自作する字句解析器によって`Token`に分解されたあと、`Statement`あるいは`Expression`インタフェースを満たす`Node`の派生オブジェクトとしてパースされていく。

```go
// Node is base of AST.
type Node interface {
	TokenLiteral() string
	String() string
}

// Statement is base statement.
type Statement interface {
	Node
	statementNode()
}

// Expression is base expression.
type Expression interface {
	Node
	expressionNode()
}
```

本書の構成のすごいところは1章でREPLを作成しコンソールで触りながら実装を進められるところだ。節の実装をひとつ終えるたびにインタプリタ（REPL）に新しい機能が「動かせる状態」で追加されていく。普段の開発で例えるならスプリントが終わるたびに必ず「新機能のデモができる」ような進め方で章が進んでいく。つまり「今は作らないけど今後の拡張を考慮しておく」アプローチについても学習することができる。前半でパーサーのコアロジックまで実装したあとは新しいトークンリテラルを追加して条件分岐に処理を追加していくだけだ。このような優れた設計・抽象化は一度実装したあと（全部一度見通したあと）のリファクタリングで得られるものなのかもしれないが、コア部分の設計が大きく開発に影響することを感じることができた。

## テスト駆動開発（TDD）で機能を着実に実装していく
本書が良いと感じた2点目の理由は最初から最後まで一貫してテスト駆動開発で実装が進むところだ。本書の各節は以下の流れで進む。

1. その節で新たに対応する命令文やリテラルの概要
2. 実装後に処理できるようになる文字列をテストケースにしたテストコードの作成
3. テストレッドを確認したあと実装を行う

特にテストコードの作成では簡単な確認のテスト以外ではすべてテーブル駆動テストで実装される。そのためテストも文字列と期待値のテストケースを数行（数ケース＿加えるだけですぐ実装が開始できる。例えば以下のテストケースにコメントアウト部分のテストケースを足せばもう`==`と`!=`の実装を開始することができる。


```go
	infixTests := []struct {
		input      string
		leftValue  interface{}
		operator   string
		rightValue interface{}
	}{
		{"5 + 5;", 5, "+", 5},
		{"5 - 5;", 5, "-", 5},
		{"5 * 5;", 5, "*", 5},
		{"5 / 5;", 5, "/", 5},
		{"5 > 5;", 5, ">", 5},
		{"5 < 5;", 5, "<", 5},
		{"5 == 5;", 5, "==", 5},
		{"5 != 5;", 5, "!=", 5},
		{"foobar + barfoo;", "foobar", "+", "barfoo"},
		{"foobar - barfoo;", "foobar", "-", "barfoo"},
		{"foobar * barfoo;", "foobar", "*", "barfoo"},
		{"foobar / barfoo;", "foobar", "/", "barfoo"},
		{"foobar > barfoo;", "foobar", ">", "barfoo"},
		{"foobar < barfoo;", "foobar", "<", "barfoo"},
//	{"foobar == barfoo;", "foobar", "==", "barfoo"},
//	{"foobar != barfoo;", "foobar", "!=", "barfoo"},
//	{"true == true", true, "==", true},
//  {"true != false", true, "!=", false},
//  {"false == false", false, "==", false},
	}
```

テストを回しながらリグレッションテストを繰り返すだけで「実装の完了」を明確に確認できる。「デグレなく実装できたか」、「今どこまで実装できているのか（どの構文まで対応できているのか）」を確かめながら新たな実装に進むことができるので、テスト駆動開発の強さを体感しながら読み進めることができた。「課題」として取り組みたい人はテストケースを実装したところで本を閉じ、自分で考えて実装するという進め方もできるだろう。

- TableDrivenTests
  - https://github.com/golang/go/wiki/TableDrivenTests

# 終わりに
本書ではインタプリタを作るという目的の他に以下の点を学習することができた。

- 優れた設計は拡張性が高く、「小さく作る」、「常に動くものを作る」を可能にする。
- テスト駆動開発とテーブル駆動テストを用いることで検証を繰り返しながら着実に開発を進めることができる。

正直に言うと私は[途中までは読書と一緒にコードも実装していた](https://github.com/budougumi0617/waiig)のだが、時間の関係（[Goによる並行処理](https://www.oreilly.co.jp/books/9784873118468/)が発売されたのもある）で中盤以降は読むだけで終えてしまった。ただこの本は何周かする予定なので二週目は実装の方も再開したい。

また、これは本の内容にはまったく関係がないし本の構成の問題ではないのだが、自分の英語能力の低さを強く感じた。少し時間を開けて読書を再開するたびに遡ってインターフェースや構造体を定義したページに戻らないと、今拡張しているオブジェクト・実装しているインターフェースの「意味」を名前から連想できていなかった。例えば`InfixExpression`とは何を定義しているのだろうか？`Expression`と`Statement`という英単語が持つ意味合いの違いはなんだろうか？となっていた。
これはフレームワークや標準パッケージを利用する際、仕様を読む際、命名付けをする際、日常に頻繁に直面する問題なのでもっとしっかり英語の勉強もしないといけないと痛感した。

なお、この本には続編の"Go言語で作るコンパイラ"もある（2018/10/30現在翻訳版はなし）。いずれこちらも読んでみたい。

- Writing A Compiler In Go
  - https://compilerbook.com/

{{< amazon category="amazon" key="interpreter" >}}
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873118220&linkId=ce2985386081de40a6b08a555d61ebe0"></iframe>
