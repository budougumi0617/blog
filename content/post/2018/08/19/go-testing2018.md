+++
title = "Goのtestを理解する in 2018 #go"
date = 2018-08-19T12:14:01+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "test", "coverage"]
+++

2018年夏(Go1.10)時点でGoのテスト方法をまとめる。

<!--more-->

# TL;DR
- Goでテストを行なうときの方法をまとめた。
- 原則標準パッケージ・標準コマンドの説明のみ
- サンプルリポジトリは以下
  - https://github.com/budougumi0617/go-testing

# testing パッケージ
https://golang.org/pkg/testing/

まずは基本として`testing`パッケージがある。Goのテストで使うメソッドは基本的にこのpkg配下にある。（HTTP関連のテストで使う`httptest`は`net/http`pkg配下にある。）  
Goでテストを書くときは`testing`パッケージを使う。

テストは`xxx_test.go`というファイルで作成する。  
この命名規則を守ることで`go build`のときは無視され、`go test`のときだけビルドされる。

テストメソッドは`*testing.T`型を引数として、`TestXxx`という命名規則で書かれる。`Test`のあとの単語もキャメルケースで命名する必要がある。`TestSum`は実行されるが、`Testsum`は実行されない。

```go
func TestSum(t *testing.T) {
	a, b, want := 1, 2, 3
	if got := Sum(a, b); got != want {
		t.Fatalf("want = %d, got = %d", want, got)
	}
}
```

構造体メソッドのテストを書くときは`TestSturct_Method`という命名するのが望ましい(`Examples`を書くときとフォーマットを合わせるため)


# テストを実行する
テストの書き方の前にテストの実行方法をまとめる。テストは`go test`コマンドで実行する。

## パッケージを指定して実行する
`go test`コマンドは以下のように実行できる。

```bash
# カレントパッケージのテストを実行する
$ go test

# パスで指定されたパッケージのテストを実行する(相対パス指定が可能)
$ go test ./table

# サブパッケージを含めたパスで指定されたパッケージ以下のテストを全て実行する
$ go test ./...
```

## Go1.10から事前に`go vet`コマンドが実行されている
Go1.10からは`go test`が実行される前に`go vet`が実行される。  
どうしても`go vet`を通さずテストの結果をみたい場合は`go test -vet=off`とする。

- CL74356 cmd/go: run vet automatically during go test
  - https://go-review.googlesource.com/c/go/+/74356

- Go 1.10 ツール周辺の CL を読む
  - https://docs.google.com/presentation/d/1FXFve72MrTfWOyj4GRERRN1bY4eivymqx4Bojq7wQuQ/edit#slide=id.p

## Go1.10からテスト結果がキャッシュされている
Go1.10から`go build`の結果がキャッシュされるように、`go test`の結果もキャッシュされる（つまり実行されない。特定のオプション時は除外）。  
キャッシュを削除したいときは`go clean -testcache`コマンドでキャッシュを削除する。  
キャッシュを使わずにテストを実行するだけならば`-count=1`オプションをつけて`go test`を実行する。

```bash
# t/parallel パッケージの結果だけキャッシュされている
$ go test ./...
ok  	github.com/budougumi0617/go-testing/t	0.009s
ok  	github.com/budougumi0617/go-testing/t/parallel	(cached)
ok  	github.com/budougumi0617/go-testing/t/table	0.009s
# 全ての結果がキャッシュされている（つまり何も実行していない）
$ go test ./...
ok  	github.com/budougumi0617/go-testing/t	(cached)
ok  	github.com/budougumi0617/go-testing/t/parallel	(cached)
ok  	github.com/budougumi0617/go-testing/t/table	(cached)
# キャッシュを使わずに全て実行する
$ go test ./... -count=1
ok  	github.com/budougumi0617/go-testing/t	0.011s
ok  	github.com/budougumi0617/go-testing/t/parallel	0.011s
ok  	github.com/budougumi0617/go-testing/t/table	0.010s
```

- How Go cache tests
  - https://speakerdeck.com/timakin/how-go-cache-tests

- Go 1.10 Release Notes
  - https://golang.org/doc/go1.10#test



## 特定のテストファイルだけを実行する
同じパッケージのメソッドを対象にした特定のテストファイルを明示的に指定してテストする場合は、テスト対象のファイルも引数に指定する。
```bash
# sum.goの中にテストするメソッドが含まれている
$ go test sum_test.go sum.go
```

後述する`package XXX_test`を使ったテストの外部パッケージ化をしているときは必然的に`import`しているのでテストファイルだけで実行できる。

## 特定のテストケースだけを実行する
特定のテストケースだけ実行する場合は`-run`オプションで指定することができる(正規表現可)。

```bash
# カレントパッケージ配下にある"Minus"というサブテスト(後述）だけを実行するとき
$ go test ./... -v -run /Minus
```

## `-parallel n`オプションで最大平行実行数を制御する
後述する`T.Parallel()`メソッドが呼ばれているテストケースは並行に実行される。  
同時に実行されるテストケースの数はデフォルトだと`GOMAXPROCS`の数と等しい。  
`-parallel n`オプションを使うと、`n`の数で同時実行数を制御できる。

## `-cpu list`オプションで`GOMAXPROCS`を変更しながらテストする
`-cpu`オプションは実行時の`GOMAXPROCS`の数を変更することができる。  
オプション引数には数字の羅列を渡すことができ、例えば`-cpu 1,2,4`とすると`GOMAXPROCS`を変えながら都合3回テストが実行される。

- go testの並列（-cpuと-parallel）がなんの事だったか忘れた時のメモ #golang
  - https://qiita.com/tenntenn/items/2832149c6fca15b66397

## デバッガ(`delve`)でテストを実行する
Delveでテストのデバッグ実行をするときは以下のようにオプションを渡す。  
普段`-run`と指定していても、正式なオプションは`-test.run`などだったりする。

```bash
$ dlv test -- -test.run TestCreateTempFile
```

- [Goのデバッガ(Delve)のいろいろな起動のしかた](/2018/04/08/debug-by-delve/)

## CI上でテストを実行する
JUnit形式で結果を出力するとCIの機能でテスト結果を可視化してくれたりする。  
CircleCIの場合は公式にやり方が書いてある。

```yaml
      - run:
          name: Run unit tests
          command: |
            trap "go-junit-report <${TEST_RESULTS}/go-test.out > ${TEST_RESULTS}/go-test-report.xml" EXIT
            go test -v ./... | tee ${TEST_RESULTS}/go-test.out
```

サンプルリポジトリでCircleCIを実行した結果はこちら

- https://circleci.com/gh/budougumi0617/go-testing

- Language Guide: Go
  - https://circleci.com/docs/2.0/language-go/

## playgroundでテストを実行する
ちょっと前からPlaygroundでもテストメソッドを簡単に実行できるようになっている。  
`main()`メソッドを宣言せずにコードを書けばよい。

https://play.golang.org/p/Timl696Usih


- The Go Playground で Test & Example がしやすくなった
  - https://qiita.com/cia_rana/items/b6bd6dd9f5af3f95ea2a

# テストの書き方
## テストコードのパッケージ名について

Goは同じディレクトリ以下に複数の`package`名を許さないが、例外的に`xxx`パッケージと`xxx_test`パッケージは同じディレクトリに共存できる。  
当然異なるパッケージになるので、非公開メソッドなどは参照できなくなるが、循環参照を回避できる。  
また、非公開な機能を`export_test.go`を介してアクセスすることもできる。

- Go Fridayこぼれ話：非公開（unexported）な機能を使ったテスト #golang
  - https://tech.mercari.com/entry/2018/08/08/080000

## `testing.TB`インターフェースの主なメソッド
通常のテスト用の`testing.T`とベンチマーク用の`testing.B`は`Errorf`などいくつか共通メソッドを持っている。  
それらは`testing.TB`インターフェースのメソッドとして定義されている。Goのテストはここで定義されているメソッドを使ってテストの正否を判定する。

- `TB.Error`/`TB.Errorf`はテストの失敗が記録されるが、後続処理は継続される
- `TB.Fatal`/`TB.Fatalf`はテストの失敗が記録されテストケースは終了する。その場で宣言済みの`defer`処理なども開始される
- `TB.Fail`はテストが失敗したことが記録されるが、その後の処理は継続される。`TB.Error`などが内部的に呼んでいる
- `TB.FailNow`はその場でテストケースが終了し、`defer`処理などが開始される。`TB.Fatal`などが内部的に呼んでいる
- `TB.SkipNow`/`TB.Skip`はテストがその場でスキップされる。テストケースを一時的に無効にしたいときに使う。
- `TB.Log`/`TB.Logf`は`-v`オプションがついていたとき、テストがFailしたときに出力される。ベンチマーク時は常に出力される。


## `testing.TB.Helper`メソッド
テストケースを複数書いていると、共通処理を外出ししてサブメソッドにすることがしばしばある。  
（たとえば構造体を走査しててエラーを判断したり）  
そうすると、エラーログがサブメソッド内になってしまう。Go1.9からそれを防ぐ`TB.Helper`メソッドが追加された。  
サブメソッド内で同メソッドを呼ぶと、サブメソッド内でFailしても呼び出し元メソッドの情報が出力される。  


何もしないサブメソッドと、`TB.Helper()`メソッドの呼び出すサブメソッドを`helper_test.go`ファイルに記述し、
```go
package helper_test

import (
	"testing"
)

func errorf(tb testing.TB, want, got int) {
	tb.Errorf("want = %d, got = %d", want, got)
}

func errorfHelper(tb testing.TB, want, got int) {
	tb.Helper()
	tb.Errorf("want = %d, got = %d", want, got)
}
```

それぞれを`sample_test.go`ファイル内のテストケースから呼び出すと、

```go
package helper_test

import (
	"testing"
)

// Sum returns two int values.
func Sum(a, b int) int {
	return a + b
}

func TestSum(t *testing.T) {
	a, b, want := 1, 2, 4
	if got := Sum(a, b); got != want {
		errorf(t, want, got)
		errorfHelper(t, want, got)
	}
}
```

`TB.Helper()`メソッドを呼んだほうだけ呼び出し元の`sample_test.go`の情報を持ってエラーが出力される。

```bash
$ go test
--- FAIL: TestSum (0.00s)
	helper_test.go:8: want = 4, got = 3
	simple_test.go:16: want = 4, got = 3
FAIL
exit status 1
FAIL	github.com/budougumi0617/go-testing/t/helper	0.010s
```

## Table Driven Test
Goではテーブルを使ってテストを書くことが推奨されている。

- テーブルでテストを用意することで、テーブル内容を確認すれば評価済みの入出力セットをすぐ確認することができて見通しが良い。  
- また、バグが発見されたときはテーブルに「バグ発生時の入力とあるべき出力」を追加するだけでTDDがスタートできる。

```go
  func TestSum(t *testing.T) {
      tests := []struct {
          a, b, want int
      }{
          {0, 1, 1},
          {-1, -1, -2},
          {-3, 2, -1},
      }
      for _, tt := range tests {
          if got := parallel.Sum(tt.a, tt.b); got != tt.want {
              t.Fatalf("want = %d, got = %d", tt.want, got)
          }
      }
  }
```

## `for _, tt := range tests`

標準パッケージで確認すると、テーブル駆動テストをするときは`tests`で宣言してループを`tt`で流していることが多いので、それに従って書くことが多い。(`for _, test := range tests`も多いが、`test`は長いので`tt`を選ぶ。)

## Sub Test(`testing.T.Run`)
`testing.T.Run`メソッドを使うとサブテスト書くことができる。

```go
  func TestSum(t *testing.T) {
      tests := []struct {
          name       string
          a, b, want int
      }{
          {"Simple", 0, 1, 1},
          {"Minus", -1, -1, -2},
          {"Both", -3, 2, -1},
      }
      for _, tt := range tests {
          t.Run(tt.name, func(t *testing.T) {
              if got := parallel.Sum(tt.a, tt.b); got != tt.want {
                  t.Fatalf("want = %d, got = %d", tt.want, got)
              }
          })
      }
  }
```

テーブルテストとサブテストを組み合わせることで以下の効果が得られる。

- テーブルのテストケースそれぞれでテストの可否を判断することができる
- 後述する`testing.T.Parallel`メソッドを使うことで並行実行ができる
- `-test.run`オプションでサブテストそれぞれを個別に実行することができる

サブテストの名前は`testing.T.Run`の第一引数に渡した文字列になる。スペースを含んだ文字列は`_`で置換されるので、使わないほうがよい。  
「このテストケースでは何をテストしている（テストしたい）のか？」がわかりやすいようにtable要素で名前をつけたほうがテストコードの可読性が高い。

```go
tests := []struct {
  name string
	// ...
}{}

for _, tt := range tests {
	t.Run(tt.name, func(t *testing.T) {
		// ...
	})
}
```

冒頭の`TestSum`を実行すると以下のようになる。

```bash
$ go test ./... -v
=== RUN   TestSum
=== RUN   TestSum/Simple
=== PAUSE TestSum/Simple
=== RUN   TestSum/Minus
=== PAUSE TestSum/Minus
=== RUN   TestSum/Both
=== PAUSE TestSum/Both
=== CONT  TestSum/Simple
=== CONT  TestSum/Both
=== CONT  TestSum/Minus
--- PASS: TestSum (0.00s)
    --- PASS: TestSum/Simple (0.00s)
    --- PASS: TestSum/Both (0.00s)
    --- PASS: TestSum/Minus (0.00s)
PASS
ok  	github.com/budougumi0617/go-testing/t/parallel	0.009s
```

`-run`オプションで特定のサブテストだけ実行する例は以下になる。

```bash
$ go test ./... -v -run Sum/Both
=== RUN   TestSum
=== RUN   TestSum/Both
=== PAUSE TestSum/Both
=== CONT  TestSum/Both
--- PASS: TestSum (0.00s)
    --- PASS: TestSum/Both (0.00s)
PASS
ok  	github.com/budougumi0617/go-testing/t/parallel	0.011s
```

## wantとgot
`expect`や`actual`は単語が長いので、`want`と`got`で書くほうがベターに思える。

- Go testing style guide
  - https://arp242.net/weblog/go-testing-style.html


## エラーは無視しない
ある一つのメソッドのテストを一つのテーブルに書いていると、以下のようなテストも同じテーブルに入れたくなる。

- 通常ケースのテスト（エラーが出ないテスト）
- ある特定のエラーがでたことを確認するテスト

その場合はテストケース内でエラーの有無をフラグとして検証条件を増やす。

```go
  tests := []struct {
      name      string
      in        int
      want      int
      wantError bool
      err       error
  }{
      {"Basic", 4, 4, false, nil},
      {"HasError", -1, 0, true, errors.New("Negative value")},
  }
  for _, tt := range tests {
      t.Run(tt.name, func(t *testing.T) {
          pt := we.PositiveInt(tt.in)
          got, err := pt.Value()
          if !tt.wantError && err != nil {
              t.Fatalf("want no err, but has error %#v", err)
          }

          if tt.wantError && !reflect.DeepEqual(err, tt.err) {
              t.Fatalf("want %#v, but %#v", tt.err, err)
          }

          if !tt.wantError && got != tt.want {
              t.Fatalf("want %q, but %q", tt.want, got)
          }
      })
  }
```

エラーの有無だけではなく期待するエラーかも確認するのが望ましい。

## Errorfのテンプレート文字列

文字列を出力するときは`%#v`を使ったほうが良い（ときもある）。微妙に空白が混ざっていたとしても可視化できる。


```go
package main

import (
	"fmt"
)

func main() {
	spaces := "      \n "
	fmt.Printf("spaces = %v\n", spaces)  // spaces =
	fmt.Printf("spaces = %#v\n", spaces) // spaces = "      \n "
}
```

- https://play.golang.org/p/WmNrzxzq3aG

## `testing.T.Parallel`メソッドで平行実行するテストを書く
Goのテストは通常逐次的に実行される。平行実行オプションをつけて実行してもそれぞれのテストケースは逐次的にされる。  
テストケースの中で`T.Parallel()`メソッドが呼ばれているテストケースのみが平行に実行される。  
各サブテストを並行実行したい場合は、`testing.T.Run()`メソッドで呼ぶ`func(*testign.T)`メソッド内で呼ぶ。

```go
  for _, tt := range tests {
      tt := tt // Don't forget when parallel test
      t.Run(tt.name, func(t *testing.T) {
          t.Parallel()
          if got := Sum(tt.a, tt.b); got != tt.want {
              t.Fatalf("want = %d, got = %d", tt.want, got)
          }
      })
  }
```

ループ変数の`tt`をローカル変数で補足するのを忘れないこと。

- 初級者向けGoの落とし穴と解説 - ループ変数の補足
  - https://speakerdeck.com/budougumi0617/traps-and-explanations-in-go?slide=24


# `TestMain(m *testing.M)`メソッドで前処理を定義する
パッケージのテストを実行する前後で任意の処理を実行してからテストを行いたいときは`TestMain(m *testing.M)`を宣言しておく。  
`m.Run()`前後に処理を記述することで、そのパッケージのテストの実行前後に処理を割り込ませることができる。

```go
func TestMain(m *testing.M) {
	func() {
		fmt.Println("Prepare test")
	}()
	ret := m.Run()
	func() {
		fmt.Println("Teardown test")
	}()
	os.Exit(ret)
}
```

```bash
$ go test
Prepare test
PASS
Teardown test
ok  	github.com/budougumi0617/go-testing/t/testmain	0.009s
```

当然(?)だが、特定のテストファイルのみを引数に実行した場合は`TestMain`は実行は実行されない。

```bash
# TestMainはmain_test.goに含まれている
$ go test sumsub_test.go -v
=== RUN   TestSum
...
    --- PASS: TestSub/Both (0.00s)
PASS
ok  	command-line-arguments	0.011s

# TestMainを含んだmain_test.goと一緒に実行する
go test sumsub_test.go main_test.go -v
Prepare test
=== RUN   TestSum
...
    --- PASS: TestSub/Both (0.00s)
PASS
Teardown test
ok  	command-line-arguments	0.009s
```

- Goでテストを書く(テストの実装パターン集)  - パッケージ全体でSetup/Teardownを使う
  - https://qiita.com/atotto/items/f6b8c773264a3183a53c

# ビルドタグでテストケースを分類する
`go build`と同じように、`go test`もビルドタグを使うことができる。  
テストファイルごとに適切なビルドタグを指定しておけば、ビルドタグが有効なときだけ実行されるテストケースを宣言しておくことができる。

`-tags integration`オプションが付与されたときだけ有効なテストファイルを作成し、

```go
// +build integration

package tags_test

import (
	"testing"

	"github.com/budougumi0617/go-testing/t/tags"
)

func TestSub(t *testing.T) {
	// some testing...
}
```

通常のファイルは`-tags integration`オプション有効時に無効にしておけば、

```go
// +build !integration

package tags_test

import (
	"testing"

	"github.com/budougumi0617/go-testing/t/tags"
)

func TestSum(t *testing.T) {
	// some testing...
}
```

複数目的のテストを分類して実行できる。

```bash
$ go test -v
=== RUN   TestSum
--- PASS: TestSum (0.00s)
PASS
ok  	github.com/budougumi0617/go-testing/t/tags	0.009s

$ go test -v -tags integration
=== RUN   TestSub
--- PASS: TestSub (0.00s)
PASS
ok  	github.com/budougumi0617/go-testing/t/tags	0.009s
```

# テストカバレッジを集計する
Goは標準機能でテストカバレッジを計測することができる。  
Go1.10からは複数パッケージのカバレッジも簡単に取得できるようになった。

単純にコマンドライン上に計測結果を出力したいならば`go test`コマンドに`-cover`オプションをつけて実行すればよい。
```bash
$ go test -cover ./...
```
## HTMLにカバレッジを保存する
`go tool`と組み合わせるとカバレッジの計測結果を
`-covermode=count`(`atomic`)オプションをつけておくと各コード行の実行回数も計測できる。

```bash
$ go test ./... -covermode=count -coverprofile=c.out
$ go tool cover -html=c.out -o coverage.html
```

## CIと組み合わせる

たとえばCircle CIならばCI中の成果物を保存しておく設定があるので、これで前述のHTMLファイルを保存しておく。

```yml
- store_artifacts:
    path: /code/test-results
    destination: prefix
```


Codecovと連携するならばCI中で以下のようにスクリプトを呼ぶだけで結果を送信できる。

```bash
$ go test ./... -coverprofile=coverage.txt -covermode=count
$ bash <(curl -s https://codecov.io/bash)
```

サンプルリポジトリを実行したCodecovの結果はこちら。

- https://codecov.io/gh/budougumi0617/go-testing



- Configuring CircleCI
  - https://circleci.com/docs/2.0/configuration-reference/#store_artifacts
- Codecov Go Example
  - https://github.com/codecov/example-go


# 関連
TODO ベンチマーク、Examples、httptestなどを別記事にまとめる。
