+++
title= "Goのtestを理解する in 2019"
date= 2019-10-30T18:08:13+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "test", "testing"]
keywords = ["Go", "golang", "testing", "test"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

昨年Go1.10時点でのGoのテストについてまとめた。  

- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018/)

まとめ記事を書いた後にリリースされたGo1.11からGo1.13に含まれるテスト関連の変更をまとめる。


<!--more-->

# TL;DR
- Go1.10までのテスト関連の機能、基本的な書き方は以下にまとめてある
  - [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018/)
  - [Goのtestingを理解する in 2018 - Examples編 #go](/2018/08/30/go-testing2018-examples/)
  - [Goのtestingを理解する in 2018 - quickサブパッケージ編 #go](/2018/09/05/go-testing2018-quick/)
  - [Goのtestingを理解する in 2018 - iotestサブパッケージ編 #go](/2018/09/09/go-testing2018-iotest/)
- Go1.11に含まれるテスト関連の変更は以下の通り
  - `go test`実行時の`go vet`の挙動の修正
  - `-memprofile`オプションがデフォルトでアロケートしたメモリを記録するようになった
- Go1.12に含まれるテスト関連の変更は以下の通り
  - `map`型を`fmt`で出力したときの結果がソートされた状態になった
  - `-benchtime`オプションで回数を指定できるようになった
- Go1.13に含まれるテスト関連の変更は以下の通り
  - `go getコマンドに`-t`オプションが追加された
  - `testing.B.N`フィールドの結果が丸まらないようになった
  - `testing.Init`関数が追加された
  - `testing.B.ReportMetric`メソッドを使って時前定義のベンチマークが計算できるようになった

# Go1.10までのテストの書き方・基本
Go1.10までのテスト関連の機能、基本的な書き方は以下にまとめてある。Go1.Xの仕様は安定しているし、Goのテストは標準パッケージの`testing`パッケージを使うだけなので何も変わっていない。

- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018/)
- [Goのtestingを理解する in 2018 - Examples編 #go](/2018/08/30/go-testing2018-examples/)
- [Goのtestingを理解する in 2018 - quickサブパッケージ編 #go](/2018/09/05/go-testing2018-quick/)
- [Goのtestingを理解する in 2018 - iotestサブパッケージ編 #go](/2018/09/09/go-testing2018-iotest/)

# Go1.11に含まれるテスト関連の変更

- Go 1.11 Release Notes
  - https://golang.org/doc/go1.11

## `go test`実行時の`go vet`の挙動の修正
Go1.10から`go test`実行時、テストが実行される前に`go vet`コマンドが実行される。`go vet`でチェック出来ていなかった`unused variable`エラーが検知できるようになった。

## `-memprofile`オプションがデフォルトでアロケートしたメモリを記録するようになった
ベンチマークテストのオプションの一つ、`-memprofile`オプションでテストの開始時からアロケートしたメモリ量を計測できるようになった。
次のようにオプションを指定してベンチマークテストを実行する。


```bash
$ go test -bench . -memprofile mem.out
goos: darwin
goarch: amd64
pkg: github.com/budougumi0617/gopl/ch11/ex06
BenchmarkPopCountByTable10-4             3235377               313 ns/op
BenchmarkPopCountByClear10-4            34185973                32.5 ns/op
...
PASS
ok      github.com/budougumi0617/gopl/ch11/ex06 26.001s
```

実行後、取得した`mem.out`を利用すると、ベンチマーク中のメモリ利用状況がわかる。

```bash
$ go tool pprof -alloc_objects ex06.test mem.out
File: ex06.test
Type: alloc_objects
Time: Oct 30, 2019 at 8:34pm (JST)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 144, 100% of 144 total
Showing top 10 nodes out of 14
      flat  flat%   sum%        cum   cum%
       144   100%   100%        144   100%  regexp.(*bitState).reset
         0     0%   100%        144   100%  main.main
         0     0%   100%        144   100%  regexp.(*Regexp).MatchString
...
```

# Go1.12に含まれるテスト関連の変更

- Go 1.12 Release Notes
  - https://golang.org/doc/go1.12

## `map`型を`fmt`で出力したときの結果がソートされた状態になった
Goのデータ構造はその定義に忠実だ。`map`はいわゆる集合なので、順序は保証されない。
そのため、Go1.11まで`map`の中身を`range`で取り出すと毎回違う順で中身が取り出されたり、`fmt`パッケージで出力すると内部データが順不同で出力されていた。
Go1.12からは`fmt`パッケージで出力する場合は常にソートされた状態で出力されるようになった。
これによって、Go1.11までで書くのが難しかった`map`の出力結果を比較するExamples Testが書きやすくなった。

- https://play.golang.org/p/MwiLpzc4xtY

```go
package main

import (
  "fmt"
)

func ExampleSalutations() {
  m := map[string]string{
    "hoge": "value1",
    "foo":  "value2",
  }
  fmt.Println(m)
  // Output:
  // map[foo:value2 hoge:value1]
}
```

「そもそもExamplesとは？」という方は次の記事を参考にしていただきたい。

- [Goのtestingを理解する in 2018 - Examples編 #go](/2018/08/30/go-testing2018-examples/)

## `-benchtime`オプションで回数を指定できるようになった
Goのテストは`go test -bench .`のように書くとベンチマークテストを実行することができる。
ここで`-benchtime Xs`オプションで秒数`Xs`を渡すと、通常1秒間で行なうベンチマークテストを任意の`X`秒間に変更することができる。Go1.12からは`-benchtime 10x`というように`数字x`を引数に渡すことで秒数指定ではなく、回数指定でベンチマークテストを実行できるようになった。

```bash
$ go test -bench BenchmarkPopCountByTable10 -benchtime=2s
goos: darwin
goarch: amd64
pkg: github.com/budougumi0617/gopl/ch11/ex06
BenchmarkPopCountByTable10-4             7659824               305 ns/op
BenchmarkPopCountByTable100-4            3503498               680 ns/op
BenchmarkPopCountByTable1000-4            550845              4344 ns/op
BenchmarkPopCountByTable10000-4            58653             39997 ns/op
PASS
ok      github.com/budougumi0617/gopl/ch11/ex06 12.994s

$ go test -bench BenchmarkPopCountByTable10 -benchtime=10x
goos: darwin
goarch: amd64
pkg: github.com/budougumi0617/gopl/ch11/ex06
BenchmarkPopCountByTable10-4                  10               729 ns/op
BenchmarkPopCountByTable100-4                 10              1276 ns/op
BenchmarkPopCountByTable1000-4                10              7114 ns/op
BenchmarkPopCountByTable10000-4               10             60297 ns/op
PASS
ok      github.com/budougumi0617/gopl/ch11/ex06 0.015s

```

# Go1.13に含まれるテスト関連の変更

- Go 1.13 Release Notes
  - https://golang.org/doc/go1.13

## `go getコマンドに`-t`オプションが追加された
`go get -u`コマンドを実行しても、通常テストに関連する依存パッケージの更新は行われない。
Go1.13からは`go get`コマンドに`-t`オプションが追加され、テストで依存しているパッケージも更新できるようになった。

```bash
$ go help get | grep "\-t" -A 1
usage: go get [-d] [-f] [-t] [-u] [-v] [-fix] [-insecure] [build flags] [packages]

--
The -t flag instructs get to also download the packages required to build
the tests for the specified packages.
```
## `testing.B.N`フィールドの結果が丸まらないようになった
Go1.12以前のベンチマークテストでは、ベンチマークテスト実行時の試行回数が丸められていた。
Go1.13からベンチマークテスト実行時の試行回数として厳密な回数が表示されるようになった。

## `testing.Init`関数が追加された
Go1.13からテスト実行時のフラグの処理順序が変更された。
そのため、テストコードの中でフラグのパースをしている場合は、フラグのパースの前に明示的に`testing.Init`関数を呼ばないとうまく実行できないようになった。
詳細は[@ktr](https://twitter.com/ktr_0731)さんのブログを参照のこと。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fsyfm.hatenablog.com%2Fentry%2F2019%2F10%2F01%2F063436" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

- https://play.golang.org/p/jPCg5PwWF-H

```go
package main

import (
  "flag"
  "testing"
)

var (
  myflag = flag.Bool("myflag", false, "custom flag")
)

func init() {
  // testing.Init() // コメントアウトを外さないとエラー
  flag.Parse()
}

func TestHello(t *testing.T) {
}

/*
 * 実行結果
 * flag provided but not defined: -test.v
 * Usage of NaClMain:
 *   -myflag
 *      custom flag
 */

```

## `testing.B.ReportMetric`メソッドを使って時前定義のベンチマークが計算できるようになった
Goの標準のベンチマークテストを実行すると、既定の計測基準でベンチマークスコアが測定される。

```bash
$ go test -bench . -benchmem
goos: darwin
goarch: amd64
pkg: github.com/budougumi0617/gopl/ch11/ex06
BenchmarkPopCountByTable10-4             3295702               311 ns/op               0 B/op          0 allocs/op
BenchmarkPopCountByClear10-4            36390008                32.5 ns/op             0 B/op          0 allocs/op
...
```

Go1.13からはこれらに加えて自分で定義したカスタムスコアを設定することでできるようになった。

```go
func BenchmarkSort(b *testing.B) {
  var cmps int64

  // ベンチマークテストを書く

  // 独自のベンチマーク基準を設定しておく
  b.ReportMetric(float64(cmps)/float64(b.N), "cmps/op")
}
```

例えば、外部リソースを模したモックなどにカウンタを設置し、何回呼び出されているかを計測するなど、色々考えられる。
詳細は[@ymotongpoo](https://twitter.com/ymotongpoo)さんの資料などを参照のこと。

- testing.B.ReportMetric @Go1.13 Release Party
  - https://bit.ly/20190823-go113-release-party
- testing.B.ReportMetric | commaok.xyz
  - https://commaok.xyz/post/report-metric/

# 終わりに
さくっと1時間くらいでまとめられるかなと思って始めたが、英語がちゃんと読めなかったりベンチマークらへんをあまり理解できていないせいでだいぶ時間がかかった。
私自身あまりベンチマークやプロファイリングなどを取ったことないので、改めて今度基礎から`pprof`などをいじってみよう。


# 参考
- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018/)
- [Goのtestingを理解する in 2018 - Examples編 #go](/2018/08/30/go-testing2018-examples/)
- [Go 1.11 Release Notes](https://golang.org/doc/go1.11)
- [Go 1.12 Release Notes](https://golang.org/doc/go1.12)
- [Go 1.13 Release Notes](https://golang.org/doc/go1.13)
- [Goのプロファイラを使ってメモリ割り当て回数を減らす](https://hnakamur.github.io/blog/2017/09/14/reduce-memory-allocations-using-go-profiler/)
- [Go 1.13 にアップデートするとテスト時に "flag provided but not defined" エラーが発生するケース](https://syfm.hatenablog.com/entry/2019/10/01/063436)
- [testing.B.ReportMetric | Go1.13 Release Party](https://bit.ly/20190823-go113-release-party)
- [testing.B.ReportMetric | commaok.xyz](https://commaok.xyz/post/report-metric/)
