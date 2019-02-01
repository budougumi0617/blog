+++
title= "golang.org/x/tools/go/analysisでLinterツールを自作する #gounco #golang"
date= 2019-02-01T08:48:38+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "ast", "analysis"]
keywords = ["Go", "AST", "静的解析"]
twitterImage = "logos/Go-Logo_Blue.png"
+++

この記事ではGo(Un)Conferenceで発表したGoにおけるLinterツールの作成方法をまとめる。

- Go(Un)Conference（Goあんこ）LT大会 5kg
  - https://gounconference.connpass.com/event/112942/
- golang.org/x/tools/go/analysisで静的解析ツールを自作する #gounco
  - https://speakerdeck.com/budougumi0617/how-to-create-the-static-analysis-tool-for-go
<!--more-->



# TL;DR
- 2018年末にGoのLinterツール用ライブラリが刷新された
  - https://godoc.org/golang.org/x/tools/go/analysis
  - モジュールとして静的解析ルールを作成・自由に組み合わせることができるようになった
- `golang.org/x/tools/go/analysis`パッケージを使うと自作Linterツールが簡単に作れる
  - https://godoc.org/golang.org/x/tools/go/analysis
  - ざっくり言うと静的解析ロジックを実装し、`Reportf`メソッドで結果を出力するだけ
  - 他の`Analyzer`の結果を利用したりもできる
- テストも直感的に書くことができる
  - https://godoc.org/golang.org/x/tools/go/analysis/analysistest
  - テスト用のGoファイルを作ってそれを読み込む仕組みが提供されている
- Linterを自作するときはtenntennさんのツールを使うととても簡単
  - https://github.com/tenntenn/gosa/tree/master/skeleton

実際に静的解析コマンドを実装したサンプルリポジトリは以下になる。

<a href="https://github.com/budougumi0617/importcheck"><img src="https://github-link-card.s3.ap-northeast-1.amazonaws.com/budougumi0617/importcheck.png" width="460px"></a>

# golang.org/x/tools/go/analysisとは
- https://godoc.org/golang.org/x/tools/go/analysis

`golang.org/x/tools/go/analysis`パッケージはGoで静的解析ツールを作成するための仕組みを提供するパッケージだ。詳細については[@tenntenn](https://twitter.com/tenntenn)さんがmercariのブログにまとめてくれている。

- [Goにおける静的解析のモジュール化について - Mercari Engineering Blog](https://tech.mercari.com/entry/2018/12/16/150000)

このパッケージを使うと静的解析ロジックをモジュール化することができ、自由に組み合わせたりすることができる。
利用例でみると、すでに`go vet`コマンドの静的解析ロジックは分割・モジュール化され、`analysis/passes`以下に各モジュールとして配置されている。
モジュールは[analysis.Analyzer](https://godoc.org/golang.org/x/tools/go/analysis#Analyzer)構造体を実装することで構成される。
`go vet`コマンドの`main`関数は以下のようになっており、各モジュールの`Analyzer`オブジェクトを読み込んでいるだけなのがわかるはずだ。

- https://github.com/golang/tools/blob/master/go/analysis/cmd/vet/vet.go


```go
// https://github.com/golang/tools/blob/master/go/analysis/cmd/vet/vet.go

func main() {
	multichecker.Main(
		// the traditional vet suite:
		asmdecl.Analyzer,
		assign.Analyzer,
		atomic.Analyzer,
		atomicalign.Analyzer,
		bools.Analyzer,
		buildtag.Analyzer,

		// ...

		nilness.Analyzer,
	)
}
```


# 静的解析モジュールを作成するための基本的な知識
[Mercariのブログ]((https://tech.mercari.com/entry/2018/12/16/150000))と[GoDoc](https://godoc.org/golang.org/x/tools/go/analysis)を見てもらえばいいのだが、`golang.org/x/tools/go/analysis`パッケージで静的解析モジュールを作る際には以下の要点を押さえておけばよい。

## Analyzer
- https://godoc.org/golang.org/x/tools/go/analysis#Analyzer

各静的解析のモジュールの実態が`Analyzer`構造体となる。`Analyzer`構造体の`Run`メソッドに静的解析ロジックを実装することで静的解析が実現する。`Analyzer`では他の`Analyzer`に解析結果を提供する際の`ResultType`、依存する他の`Analyzer`を指定する`Requires`などのフィールドが存在する。


```go
type Analyzer struct {
    // ...

    // ロジック本体。戻り値はなくても良いが、Resultとして他のAnalyzerに結果を渡す場合は返す。
    Run func(*Pass) (interface{}, error)

    // ...

    // このAnalyzerが実行時に利用したい他のAnalyzer
    Requires []*Analyzer

    // このAnalyzerが他のAnalyzerに提供する情報。Runメソッドの戻り値
    ResultType reflect.Type

    // 同じAnalyzer内で解析対象のパッケージをまたいで解析情報を利用したい場合はFactという概念を利用してやり取りする
    FactTypes []Fact
}
```
## Pass
- https://godoc.org/golang.org/x/tools/go/analysis#Pass

静的解析は実行時に受け取った解析対象のパッケージに対して`Analyzer`の`Run`メソッドを実行していく。
`Run`メソッドの引数として解析対象パッケージの情報を持つのが`Pass`構造体だ。

```go
type Pass struct {
    Analyzer *Analyzer // the identity of the current analyzer

    // 解析対象パッケージの静的解析用の情報
    Fset       *token.FileSet // file position information
    Files      []*ast.File    // the abstract syntax tree of each file
    OtherFiles []string       // names of non-Go files of this package
    Pkg        *types.Package // type information about the package
    TypesInfo  *types.Info    // type information about the syntax trees
    TypesSizes types.Sizes    // function for computing sizes of types

    // ...

    // 他のAnalyzerのReslutを利用したい場合はこの中から受け取る
    ResultOf map[*Analyzer]interface{}

    // Factをやりとりするときの関数
    ImportObjectFact func(obj types.Object, fact Fact) bool

    // ...
}
```

## 静的解析中に利用するデータ概念
`Analyzer`や`Pass`の説明中に出てきた3種類のデータの概念が以下だ。

- Result
  - https://godoc.org/golang.org/x/tools/go/analysis#Analyzer
  - その`Analyzer`に解析結果を提供するときに定義する
- Diagnostic
  - https://godoc.org/golang.org/x/tools/go/analysis#Diagnostic
  - CLIにしたときに出力されるエラー情報(`github.com/johndue/pkg/bar.go: 5:2: some analysis error`のような)
  - `Printf`関数の用に使える[Pass.Reportf](https://godoc.org/golang.org/x/tools/go/analysis#Pass.Reportf)メソッドがあるので、実際この構造体をそのまま扱うことはない（と思う）
- Facts
  - https://godoc.org/golang.org/x/tools/go/analysis#Fact
  - 同じAnalyzer内で解析対象のパッケージをまたいで解析情報を利用できる。



# 自作のLinterを作ってみる
ここからは実際に自作のLinterを作ってみる。まず最初は@tenntennさんが公開している`skeleten`コマンドを使うと簡単にテンプレートが生成される。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/tenntenn/gosa/tree/master/skeleton" data-iframely-url="//cdn.iframe.ly/bG7XmYw"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`mylinter`という名前で静的解析モジュールのテンプレートを作成するときは以下のように使う。

```bash
$ go get github.com/tenntenn/gosa/skeleton
$ skeleton mylinter
mylinter
├── mylinter.go  # 静的解析ロジックを書く本体
├── mylinter_test.go
└── testdata
    └── src
        └── a # テストを書くときのテストケースパッケージ
            └── a.go
```

これで静的解析ロジックを`mylinter.go`の中の`Analyzer.Run`に自作の静的解析ロジックを実装していくだけでよい。

```go
package mylinter

import (
        "go/ast"

        "golang.org/x/tools/go/analysis"
        "golang.org/x/tools/go/analysis/passes/inspect"
        "golang.org/x/tools/go/ast/inspector"
)

var Analyzer = &analysis.Analyzer{
        Name: "mylinter",
        Doc:  Doc,
        Run:  run,
        Requires: []*analysis.Analyzer{
                inspect.Analyzer,
        },
}

const Doc = "mylinter is ..."

func run(pass *analysis.Pass) (interface{}, error) {
  // 省略
}
```

このまま`Analyzer`をコマンドとして実行したい場合は以下のような`cmd/main.go`ファイルを用意すればよいだろう。

```go
// cmd/main.go
package main

import (
        "github.com/budougumi0617/mylinter"
        "golang.org/x/tools/go/analysis/multichecker"
        "golang.org/x/tools/go/analysis/passes/inspect"
)

func main() {
        multichecker.Main(
                // skeletonで生成したAnalyzerの初期コードの場合はinspect.Analyzerの結果に依存している
                inspect.Analyzer,
                mylinter.Analyzer,
        )
}

```

ここでは`multichecker.Main`関数で定義しているが、`Main`関数の実装はいくつか提供されている。
一つの`Analyzer`だけを使ったLinterなのか？複数の`Analyzer`を使ったLinterなのか？などによって使い分ける。


- https://godoc.org/golang.org/x/tools/go/analysis/singlechecker
- https://godoc.org/golang.org/x/tools/go/analysis/unitchecker
- https://godoc.org/golang.org/x/tools/go/analysis/multichecker

あとは通常どおり`go buid`すればLinterコマンドがもう利用できる。（以下はサンプルリポジトリを実行した例）

```bash
$ go build -o icheck ./cmd/main.go

# testdataディレクトリ以下のサンプルコードのimport構成が特殊なので
# GOPATHをちょっと変えているが、本来はGOPATHの再定義は必要ない
$ GOPATH=`pwd`/testdata:$GOPATH ./icheck ./testdata/src/...
/Users/shimizubudougumi0617/importcheck/testdata/src/handler/h.go:7:2: \
handler must not include "repository"

```

## ASTの探索について。
準備は簡単なのだが、実際に難しいところは目的の静的解析ロジックを組み立てるところだろう（逆にいうと本質的な部分にフォーカスできるようになっているとも言える）。
抽象構文木(AST)については@tenntennさんのQiitaの記事や[@motemen](https://twitter.com/motemen)さんのgit bookを見るとわかりやすい。

- [Qiitaで「"AST user:tenntenn"」と検索](https://qiita.com/search?q=AST+user%3Atenntenn&sort=)
- [GoのためのGo](https://motemen.github.io/go-for-go-book/)

`skeleton`で生成したコードにもふくまれているが、`inspect`パッケージを使うと簡単にASTを走査できるので`switch`文を組み立てるだけでパースを開始できる。

- https://godoc.org/golang.org/x/tools/go/analysis/passes/inspect

```go
func run(pass *analysis.Pass) (interface{}, error) {
        inspect := pass.ResultOf[inspect.Analyzer].(*inspector.Inspector)

        // 調べたいNodeの型でフィルタリングできる
        nodeFilter := []ast.Node{
                (*ast.Ident)(nil),
        }

        inspect.Preorder(nodeFilter, func(n ast.Node) {
                switch n := n.(type) {
                case *ast.Ident:
                        _ = n
                }
        })

        return nil, nil
}
```

あとはシンプルな`Analyzer`の実装を見ていくとなんとなくやり方が見えてくると思う（私はまだ見えていない）。

- https://github.com/golang/tools/blob/master/go/analysis/passes/nilfunc/nilfunc.go

# Analyzerのテストを用意する
- https://godoc.org/golang.org/x/tools/go/analysis/analysistest

`analysistest`パッケージを使うと、簡単に`Analyzer`をテストすることができる。
下記のように静的解析でエラーを出したいテスト用のサンプルコードを準備するだけだ。
エラーを出したい行に対して、`// want `と始めて期待出力文字列をコメントしておくだけでよい。

```go
// testdata/src/handler/h.go
package handler

import (
        "repository" // want "handler must not include \"repository\""
        "service"
)

...
```

あとは用意したサンプルコードを読み込んで、`Analyzer`を実行するテストケースを用意する。`skeleteon`コマンドでテンプレートを生成した場合、`xxx_test.go`にそのようなテストケースがすでに生成されている。
テストコードには「`./testdata/src`ディレクトリにある`"a"`パッケージを一緒に生成した`Analyzer`で静的解析する」テストケースがすでにあるので、`testdata/src/a/a.go`ファイルに自分が実装した静的解析ロジックで見つけたいエラーを書いておくだけだ。

```go
package mylinter_test

import (
        "testing"

        "github.com/budougumi0617/mylinter"
        "golang.org/x/tools/go/analysis/analysistest"
)

func Test(t *testing.T) {
        testdata := analysistest.TestData()
        analysistest.Run(t, testdata, mylinter.Analyzer, "a")
}
```

`skeleton`で生成したコードでそのまま`go test`すれば、サンプルコードに書かれた`// want`が満たされずFAILするのがわかる。

```bash
$ go test
--- FAIL: Test (0.03s)
    analysistest.go:311: a/a.go:4: no diagnostic was reported matching "pattern"
FAIL
exit status 1
FAIL    github.com/budougumi0617/mylinter       0.042s
```

# その他のTips
## 静的解析を実装するときに参考にしたいコード
そもそも静的解析でどんなことができるのか？あの静的解析はどのようにロジックを組めばできるのだろう？という疑問もあるはずだ。
そういう疑問を持ったときははまず`golang.org/x/tools/go/analysis/passes`以下のパッケージを読むといいだろう。`go vet`の静的解析の実装が全て`Analyzer`ベースになっている、

- https://godoc.org/golang.org/x/tools/go/analysis

`ExportObjectFact`など、`Facts`を使った実装は`golang.org/x/tools/go/analysis/passes/printf/printf`パッケージで行われている。

- https://github.com/golang/tools/blob/master/go/analysis/passes/printf/printf.go

`Result`については`golang.org/x/tools/go/analysis/passes/inspect/inspect`を参考にすれば良い（他のモジュールからの使い方については`skeleton`コマンドで生成したコードを見ればわかるだろう）。

- https://github.com/golang/tools/blob/master/go/analysis/passes/inspect/inspect.go


## テストケースの書き方について
テスト用のサンプルコードは`testdata`ディレクトリ以下に作成することになるが、`analysistest`パッケージでテストを実行するときは`testdata`ディレクトリが`GOPATH`になる。なので、たとえば`repository`パッケージを`import`する`handler`パッケージというサンプルコードを作ったとすると、ディレクトリ構成は以下のようになる。

```bash
$ tree testdata
testdata
└── src
    ├── domain
    │   └── d.go
    ├── handler
    │   └── h.go
    ├── repository
    │   └── r.go
    └── service
        └── s.go
```

```go
// testdata/src/handler/h.go

package handler

import (
        "encoding/json"
        "net/http"

        "repository" // want "handler must not include \"repository\""
        "service"
)

// Do anything...
```

## フラグ
CLIツールの場合フラグ処理を入れたくなるのが常だ。`analysis`パッケージを使った実装した場合、実行時のフラグは`Analyzer.Flags`に入って渡ってくる。具体的な呼び出し方は[Mercariのブログ](https://tech.mercari.com/entry/2018/12/16/150000)を見るとよいだろう。

```go
type Analyzer struct {
    // ...

    // Flags defines any flags accepted by the analyzer.
    // The manner in which these flags are exposed to the user
    // depends on the driver which runs the analyzer.
    Flags flag.FlagSet

    // ...
}
```



## 既存Linterツールの移行について
すでに`golang.org/x/tools/go/analysis`パッケージ以前の仕組みでLinterを作成していた方もいるだろう。
既存Linterツールの移行(マイグレーション)については[izumin5210](https://twitter.com/izumin5210)さんや[daisuzu](https://twitter.com/dice_zu)さんの記事が参考になる。

- [golang.org/x/tools/analysis を理解する #golang #golangtokyo](https://blog.izum.in/understand-understand-golang-org-x-tools-analysis-b7a3aba7ae4)
- [golang.tokyo 静的解析Dayでgolang.org/x/tools/go/analysisを使ってみた](https://daisuzu.hatenablog.com/entry/2018/12/08/225147)


# 終わりに
私は今までLinterを作るのは難しそうだなと思っていたが、`golang.org/x/tools/go/analysis`で提供されるようになった仕組みを使えば、静的解析ロジック以外の殆どを気にしなくてすむ。また静的解析ロジックそのものもヘルパーモジュールを使うことでだいぶラクに実装できる。
今回サンプルとして実装したLinterツールはハードコーディングだらけで使い物にならないので、設定ファイルから条件を読み込むようにしたりして実際の業務で使えるLinterを作ってみたいと思う。
また、私は英語がちゃんと読めないので、日本語でいろいろな情報を書いてくださっている先輩Gopherのみなさんに感謝しかない(ましてやジェネレータまで提供してくれている！)。

# 参考
- [package analysis - GoDoc](https://godoc.org/golang.org/x/tools/go/analysis)
- [Goにおける静的解析のモジュール化について - Mercari Engineering Blog](https://tech.mercari.com/entry/2018/12/16/150000)
- [モジュール化された静的解析の実装を追ってみよう #golang - Qita](https://qiita.com/tenntenn/items/65b9a39e6db09efabd74)
- [github.com/tenntenn/gosa/skeleton](https://github.com/tenntenn/gosa/tree/master/skeleton)
- [Qiitaで「"AST user:tenntenn"」と検索](https://qiita.com/search?q=AST+user%3Atenntenn&sort=)
- [GoのためのGo](https://motemen.github.io/go-for-go-book/)
- [静的解析で型を扱う #golang - Qiita](https://qiita.com/tenntenn/items/dfc112ae7bb8c9703ab9)
- [Go言語の golang/go パッケージで初めての構文解析](https://qiita.com/po3rin/items/a19d96d29284108ad442)
- [golang.org/x/tools/analysis を理解する #golang #golangtokyo](https://blog.izum.in/understand-understand-golang-org-x-tools-analysis-b7a3aba7ae4)
- [golang.tokyo 静的解析Dayでgolang.org/x/tools/go/analysisを使ってみた](https://daisuzu.hatenablog.com/entry/2018/12/08/225147)
