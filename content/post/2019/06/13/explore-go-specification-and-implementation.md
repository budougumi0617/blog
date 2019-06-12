+++
title= "GoのLanguage Specificationの特定の仕様に関わる処理系の実装を探す #golang"
date= 2019-06-13T08:10:59+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["golang", "git"]
keywords = ["go", "git log", "golang"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


Goの仕様はこのHTML1ページにまとまっている。

- The Go Programming Language Specification
  - https://golang.org/ref/spec

仕様に書いてある処理はどうやって実装されているのか調べたかった。  
Goの仕様の1文からその仕様に関係する処理系のコードを探す方法をメモしておく。  

<!--more-->

# TL;DR
- GoのLanguage Specificationに書いてある仕様の処理系の実装を探す
  - https://golang.org/ref/spec
- GoのLanguage SpecificationのHTMLはGoリポジトリの`doc`ディレクトリに入っている
  - https://go.googlesource.com/go
- Language Specificationのコミットログから仕様の追加日を知る
  - `git log -L 2569,2579:./doc/go_spec.html`
- 仕様の追加日周辺から関連コミットを検索する
  - `git log --since="2010/10/01" --until="2010/10/31" --grep "composite"`
- `master`ブランチにもう存在しないファイルのコミットログを見るときは`--`をつけて`git log`
  - `git log --oneline -- src/cmd/gc/typecheck.c`

# 仕様に関連する処理系の実装を探したい

標準関数などは公式の[packages](https://golang.org/pkg/)を見れば良いのだが、
処理系のコードを単純に`grep`で探すのは難しい。
リポジトリのコミットログから仕様に結びつく処理系の実装を探す。

今回は探してみた仕様は以下の部分の仕様だ。

- Composite literals | The Go Programming Language Specification
  - https://golang.org/ref/spec#Composite_literals

> Within a composite literal of array, slice, or map type T, elements or map keys that are themselves composite literals may elide the respective literal type if it is identical to the element or key type of T. Similarly, elements or keys that are addresses of composite literals may elide the &T when the element or key type...

```go
[...]Point{{1.5, -3.5}, {0, 0}}     // same as [...]Point{Point{1.5, -3.5}, Point{0, 0}}
[][]int{{1, 2, 3}, {4, 5}}          // same as [][]int{[]int{1, 2, 3}, []int{4, 5}}
[][]Point{{{0, 1}, {1, 2}}}         // same as [][]Point{[]Point{Point{0, 1}, Point{1, 2}}}
map[string]Point{"orig": {0, 0}}    // same as map[string]Point{"orig": Point{0, 0}}
```

必要なものはWebブラウザ、Go本体コードのリポジトリ、`git`と`grep`コマンドだけだ。

# GoのLanguage SpecificationのHTMLはGoリポジトリの`doc`ディレクトリに入っている
GitHubにもミラーされているが、本当（？）のリモートURLからGo本体のリポジトリを取得してくる。

- Go
  - https://go.googlesource.com/go

GoのLanguage SpecificationのHTMLはGo本体のリポジトリに一緒に格納されている。

まず、探したい仕様の行数を`grep`コマンドで確認する。

```bash
$ grep -n "Within a composite literal of array, slice" ./doc/go_spec.html
2569:Within a composite literal of array, slice, or map type <code>T</code>,
```

HTMLでは2569行目にあるらしい。

- [golang/go/doc/go_spec.html#L2569](https://github.com/golang/go/blob/82cf8bca9cf20297bc0edf481cc530c9b3f4bf1e/doc/go_spec.html#L2569)

この周辺のコミットログを探してみると、このコミットで仕様の1文が追加されたようだ。

```bash
$ git log --reverse  -L 2569,2579:./doc/go_spec.html
commit a12141e5f4e905045dca5dff2669b64d9b93788f
Author: Robert Griesemer <gri@golang.org>
Date:   Fri Oct 22 08:58:52 2010 -0700

    go spec: relaxed syntax for array, slice, and map composite literals

    For elements which are themselves composite literals, the type may
    be omitted if it is identical to the element type of the containing
    composite literal.

    R=r, rsc, iant, ken2
    CC=golang-dev
    https://golang.org/cl/2661041

diff --git a/doc/go_spec.html b/doc/go_spec.html
--- a/doc/go_spec.html
+++ b/doc/go_spec.html
@@ -2097,0 +2098,8 @@
+Within a composite literal of array, slice, or map type <code>T</code>,
+elements that are themselves composite literals may elide the respective
+literal type if it is identical to the element type of <code>T</code>.
+</p>
+
+<pre>
+[...]Point{{1.5, -3.5}, {0, 0}}  // same as [...]Point{Point{1.5, -3.5}, Point{0, 0}}
+[][]int{{1, 2, 3}, {4, 5}}       // same as [][]int{[]int{1, 2, 3}, []int{4, 5}}

commit 5f49456465f53f96bee03ac8cbe0d564e31576c2
Author: Russ Cox <rsc@golang.org>
Date:   Fri Dec 2 14:12:53 2011 -0500
```

Goはコミットログ中にあるURLを見れば`Gerrit`上のレビューの経過も確認できる。

- code review 2661041: go spec: relaxed syntax for array, slice, and map compo...
  - https://golang.org/cl/2661041

# 仕様が追加された日の周辺コミットログから実装の追加コミットログを探す
仕様が書き足されたのは`Fri Dec 2 14:12:53 2011`だった。この日付の周辺で実装が追加されたコミットがないか探してみる。
`git log`コマンドで日付範囲を指定し、今回の実装に関連しそうな`compoiste`をキーワードにコミットを探してみる。

```bash
$ git log --since="2010/10/01" --until="2010/10/31" --grep "composite"
commit f613015e0eeb9560579bf40dbdb40fac5e371bbc
Author: Robert Griesemer <gri@golang.org>
Date:   Fri Oct 22 10:03:14 2010 -0700

    go ast/parser/printer: permit elision of composite literal types for composite literal elements
    gofmt: added -s flag to simplify composite literal expressions through type elision where possible

    R=rsc
    CC=golang-dev
    https://golang.org/cl/2319041

commit a12141e5f4e905045dca5dff2669b64d9b93788f
Author: Robert Griesemer <gri@golang.org>
Date:   Fri Oct 22 08:58:52 2010 -0700

    go spec: relaxed syntax for array, slice, and map composite literals

    For elements which are themselves composite literals, the type may
    be omitted if it is identical to the element type of the containing
    composite literal.

    R=r, rsc, iant, ken2
    CC=golang-dev
    https://golang.org/cl/2661041

commit 8ffc4ec5d0c3d17d633c277ba5102da838834f03
Author: Russ Cox <rsc@golang.org>
Date:   Thu Oct 21 23:17:20 2010 -0400

    gc: implement new composite literal spec

    R=ken2
    CC=golang-dev
    https://golang.org/cl/2350041
```

それぞれのコミットに書かれているレビュー記録を見てみる。

- code review 2350041: gc: implement new composite literal spec
  - https://codereview.appspot.com/2350041

(Go1.4以前はGoはセルフコンパイルではなかったので、)Cの実装ファイル（`src/cmd/gc/typecheck.c`）が見つかった。

- code review 2319041: go ast/parser/printer: permit elision of composite lite
  - https://codereview.appspot.com/2319041

こちらを見るとパーサー（`src/pkg/go/parser/parser.go` etc）の関連実装を見ることができた。

## `master`ブランチにもう存在しないファイルのコミットログを見るときは`--`をつけて`git log`

現在の実装まで探すときはここからさらにコミットログをたどっていけば良い。  
`master`ブランチにもう存在しないファイルの`src/cmd/gc/typecheck.c`に関連するコミットを探したいならば、--`をつけて`git log`コマンドを実行する。


```bash
$ git log --oneline -- src/cmd/gc/typecheck.c
3af0d791be [dev.cc] cmd/6a, cmd/6g etc: replace C implementations with Go implementations
5c87cf7608 cmd/gc: minor adjustments for C to Go translation
4b27c9d72e cmd/gc: add .y to error about missing x in x.y
77a2113925 cmd/gc: evaluate concrete == interface without allocating
eaa872009d cmd/gc: fix capturing by value for range statements
0e80b2e082 cmd/gc: capture variables by value
b581ca5956 cmd/gc: allow map index expressions in for range statements
a7bb393628 cmd/gc: don't emit write barriers for *tmp if tmp=&PAUTO
8d44ede0dc cmd/gc: simplify code for c2go (more)
e82003e750 cmd/gc: simplify code for c2go
```

また、GitHubのRESTに慣れているならばアドレス直打ちで同等のコミットログが見れる。

- https://github.com/golang/go/commits/master/src/cmd/gc/typecheck.c

# 終わりに
もともとTweetでもやもやをつぶやいていた。
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">Goで ds := [][]int{{1, 2, 3, 4}} が許されるの、ラクなんだけどなんで許される（{ []int{1, 2, 3, 4} }って書かなくてもよい）のかよく分かってない<a href="https://t.co/atstRrjope">https://t.co/atstRrjope</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1138626434279268352?ref_src=twsrc%5Etfw">June 12, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

そうしたら会社の先輩が仕様の場所を教えてくれたので、せっかくだから処理系までちゃんと読んでみようと思った。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://t.co/HlOvZFgB4j">https://t.co/HlOvZFgB4j</a><br>これの後半あたりですね</p>&mdash; Yui TERASHIMA (@terashi58) <a href="https://twitter.com/terashi58/status/1138639910234017792?ref_src=twsrc%5Etfw">June 12, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

まだ、Goの実装部分まで追えていないが、長期に渡って開発されているリポジトリの`git`を辿る操作も勉強できてよかった。  
ただ、これはGoリポジトリのコミットログが異様にきれい（1PRは1コミット、かならずコミットメッセージにPRのURLが入っている）だからできることなのかもしれない。


# 参考
- [Composite literals | The Go Programming Language Specification](https://golang.org/ref/spec#Composite_literals)
- [code review 2350041: gc: implement new composite literal spec](https://codereview.appspot.com/2350041)
- [code review 2319041: go ast/parser/printer: permit elision of composite lite](https://codereview.appspot.com/2319041)
- [git-log - Show commit logs | git documentation](https://git-scm.com/docs/git-log)
- [削除したファイルの履歴を確認する](http://nafuruby.hatenablog.com/entry/2015/11/09/013901)
