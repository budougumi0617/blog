+++
title = "Goのtestingを理解する in 2018 - Examples編 #go"
date = 2018-08-30T20:49:44+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "test", "examples"]
+++

この記事は以下の記事で触れなかったExamplesについてまとめる。

- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018)

<!--more-->

# TL;DR
- 出力結果を確認するExamplesテストを書くことができる
  - `ExampleXXXX`という引数なしメソッド内に記載する
- ExamplesテストはGoDocにも一緒に出力される
  - メソッド名によってGoDocで出力される場所を制御できる
- Go1.7からはランダムな出力順序の結果もテスト出来るようになった
  - `Unordered output:`をキーワードに期待出力結果を記載する

なお、本記事に関連するサンプルリポジトリは以下。

- https://github.com/budougumi0617/go-testing

# Testable Examples
Goでは通常のテストの他に、出力を確認する`Examples`テストを書くことが出来る。

- https://blog.golang.org/examples

Examplesテストも通常のテストと同じく`xxx_test.go`というファイル内に記載する。テスト関数の名前は`Example`で始め、引数は必要ない。

```go
  // Person is sumple struct.
  type Person struct {
      Name string
      Age  int
  }

  // String returns string.
  func (p *Person) String() string {
      return fmt.Sprintf("name %s, age %d", p.Name, p.Age)
  }

  func ExamplePerson_String() {
      p := &e.Person{Name: "Alice", Age: 12}
      fmt.Println(p)

      // Output:
      // name Alice, age 12
  }
```

# Examplesケースの命名規則
Examplesのテストメソッドは以下の命名規則で宣言する。  
命名規則を守ることで、後述するようにGoDoc内の特定の場所に配置することが出来る。

|名前|配置される場所|
|---|---|
|`Example()`| パッケージのOverviewに記載される|
|`ExampleXxx()`| `Xxxx`関数、あるいは`Xxxx`構造体・インターフェースに記載される|
|`ExampleXxx_Foo()`| `Xxxx`構造体の`Foo`メソッドに記載される|
|`ExampleXxx_Foo_one()`| `Xxxx`構造体の`Foo`メソッドに複数Examplesを書きたい時|

メソッドの場合はアンダースコアで構造体名とメソッド名を結ぶ。  
特定の同じ関数（メソッド）や構造体のExamplesを書くときの識別文字は小文字で始める。

# 実行方法
Examplesテストも通常のテストケースと一緒に実行されるため、`go test`コマンドで実行できる。  
通常のテストを実行せずに、Examplesテストだけ実行したい場合は`-run`オプションを利用すればよい。

```bash
$ go test . -v
=== RUN   TestPerson
--- PASS: TestPerson (0.00s)
=== RUN   Example
--- PASS: Example (0.00s)
=== RUN   ExampleNewPerson
--- PASS: ExampleNewPerson (0.00s)
=== RUN   ExamplePerson
--- PASS: ExamplePerson (0.00s)
=== RUN   ExamplePerson_String
--- PASS: ExamplePerson_String (0.00s)
=== RUN   ExamplePerson_String_bob
--- PASS: ExamplePerson_String_bob (0.00s)
PASS
ok  	github.com/budougumi0617/go-testing/e	(cached)

$ go test . -v -run=Example*
=== RUN   Example
--- PASS: Example (0.00s)
=== RUN   ExampleNewPerson
--- PASS: ExampleNewPerson (0.00s)
=== RUN   ExamplePerson
--- PASS: ExamplePerson (0.00s)
=== RUN   ExamplePerson_String
--- PASS: ExamplePerson_String (0.00s)
=== RUN   ExamplePerson_String_bob
--- PASS: ExamplePerson_String_bob (0.00s)
PASS
ok  	github.com/budougumi0617/go-testing/e	(cached)
```

# ランダムオーダーな出力を確認する
Go1.7以降からランダムな出力結果を確認するための`Unordered output:`が追加されている。  
Goの`map`は言語仕様上イテレート時に返る結果の順序を保証していないし、実際にランダムな順序で返される。  
そのようなランダム要素が含まれる期待出力については、`Unordered output:`を使って期待出力結果を書いていおくと、順序に関係なく結果を評価できる。


```go
  func Example() {
      ps := map[int]*e.Person{
          0: &e.Person{Name: "Alice", Age: 12},
          1: &e.Person{Name: "Bob", Age: 10},
          2: &e.Person{Name: "Chris", Age: 15},
      }

      for _, p := range ps {
          fmt.Println(p)
      }

      // Unordered output:
      // name Alice, age 12
      // name Chris, age 15
      // name Bob, age 10
  }
```

# GoDocへの記載結果を確認する
ローカルでGoDocのサーバを起動すれば、ExamplesがどのようにGoDocに展開されるか確認することが出来る。
```bash
$ godoc -http=:6060
```
ローカルの`$GOPATH`以下のコード量によるが、少し待ってwebブラウザでアクセスすると、GoDocの具合を確認することができる。

![godoc](/2018/08/30_godoc.png)

なお、GoDoc自体の書き方は[渋川さん](https://twitter.com/shibu_jp)の[記事](https://qiita.com/shibukawa/items/8c70fdd1972fad76a5ce)が詳しい。

# 参考
- Testable Examples in Go
  - https://blog.golang.org/examples
- チョットできるGoプログラマーになるための詳解GoDoc
  - https://qiita.com/shibukawa/items/8c70fdd1972fad76a5ce

# 関連
- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018)


