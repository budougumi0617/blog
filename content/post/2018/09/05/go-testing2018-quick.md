+++
title = "Goのtestingを理解する in 2018 - quickサブパッケージ編 #go"
date = 2018-09-05T10:02:02+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "test"]
+++

この記事は以下の記事で触れなかった`testing/quick`パッケージについてまとめる。

- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018)
- [Goのtestingを理解する in 2018 - Examples編 #go](/2018/08/30/go-testing2018-examples/)

<!--more-->

# TL;DR
- testing/quickパッケージ
  - https://golang.org/pkg/testing/quick/
- `testing/quick`パッケージはランダムテストを行う
- 新旧メソッドを比較するデグレ確認にも使うことができる。

# quick.CheckEqual関数
- https://golang.org/pkg/testing/quick/#Check

`func Check(f interface{}, config *Config) error`は与えられた関数`f`に対してランダムテストを行う。
`f`の引数が構造体や文字列でもある程度は融通してランダムな引数を用意してくれる。
また、`f`は戻り値が`bool`である必要があるので、必要に応じてラップ関数を作る。

- https://play.golang.org/p/ovZM2kkwHBK

```go
type MyInt struct {
	V int
	X string
}

func div(n, m MyInt) int {
	return n.V / m.V
}

func TestDiv(t *testing.T) {
	// div関数は戻り値がintなので、評価結果をboolで返すラップ関数を用意する
	f := func(x, y MyInt) bool {
		return div(x, y) == (x.V / y.V)
	}

	if err := quick.Check(f, nil); err != nil {
		t.Error(err.Error())
	}
}
```

第二引数の`quick.Config`を指定すれば、試行回数や与える引数の範囲、ランダムシードなどを設定できる。

- https://play.golang.org/p/8nPw0pgUkN3

```go
	cfg := &quick.Config{
		// 試行回数
		MaxCount: 10,
		// Seed
		Rand: rand.New(rand.NewSource(2)),
		// 独自定義した引数の生成関数
		Values: func(args []reflect.Value, rand *rand.Rand) {
			args[0] = reflect.ValueOf(MyInt{rand.Int(), "x"})
			args[1] = reflect.ValueOf(MyInt{rand.Int(), "y"})
		},
	}

	if err := quick.Check(f, cfg); err != nil {
		t.Error(err.Error())
	}
```

## quick.CheckError

`quick.Check`関数が戻す`Error`は`quick.CheckError`型のポインタなので、任意の情報も取得できる。

- https://golang.org/pkg/testing/quick/#CheckError

```go
	if err := quick.Check(f, nil); err != nil {
		if ce, ok := err.(*quick.CheckError); ok {
			t.Errorf("Try count = %d, In %#v, Out %s\n", ce.Count, ce.In, ce)
		} else {
			t.Error(err)
		}
	}
```

# quick.CheckEqual
- https://golang.org/pkg/testing/quick/#CheckEqual

` quick.CheckEqual`関数は2つの関数に対して乱数テストを実施し、出力結果に変化がないか確認をできる。
Go本体のコードでは例えば`sync.Map`の挙動の確認に利用されている。

- https://github.com/golang/go/blob/67ac554d7978eb93f3dfe7a819c67948dd291d88/src/sync/map_test.go#L99-L109

```go
func TestMapMatchesRWMutex(t *testing.T) {
	if err := quick.CheckEqual(applyMap, applyRWMutexMap, nil); err != nil {
		t.Error(err)
	}
}

func TestMapMatchesDeepCopy(t *testing.T) {
	if err := quick.CheckEqual(applyMap, applyDeepCopyMap, nil); err != nil {
		t.Error(err)
	}
}
```

## quick.CheckEqualError
- https://golang.org/pkg/testing/quick/#CheckEqualError

`quick.CheckEqual`関数が戻す`Error`も`quick.CheckEqualError`として定義されているので、結果が不一致なときの各情報を取得できる。


```go
type CheckEqualError struct {
        CheckError
        Out1 []interface{}
        Out2 []interface{}
}
```

# quick.Value
ただ単にランダムな値が欲しい場合は、`quick.Value`関数で生成できる。

```go
// 乱数整数を生成する
ri, _ := quick.Value(reflect.TypeOf([]int{}), rand.New(rand.NewSource(0)))
// 乱数整数スライスを生成する
ris, _ := quick.Value(reflect.TypeOf([]int{}), rand.New(rand.NewSource(0)))
```

# Generatorインターフェース
`Generator`インターフェースを実装していれば、独自定義した型も`quick.Value`の引数に利用できる。
- https://play.golang.org/p/8HRbhNYBsnu

```go
type Foo struct{}

func (f *Foo) Generate(rand *rand.Rand, size int) reflect.Value {
	// 雑な実装
	return reflect.ValueOf(&Foo{})
}

func main() {
	v, ok := quick.Value(reflect.TypeOf([]*Foo{}), rand.New(rand.NewSource(0)))
}
```

# 参考
- GoでQuickCheckをする
  - https://ymotongpoo.hatenablog.com/entry/2012/11/30/155846
- Goでテストを書く(テストの実装パターン集) - QuickCheck(testing/quick)でブラックボックステストをする
  - https://qiita.com/atotto/items/f6b8c773264a3183a53c#quickchecktestingquick%E3%81%A7%E3%83%96%E3%83%A9%E3%83%83%E3%82%AF%E3%83%9C%E3%83%83%E3%82%AF%E3%82%B9%E3%83%86%E3%82%B9%E3%83%88%E3%82%92%E3%81%99%E3%82%8B
- Goのtesting/quickを簡単に触ってみる
  - http://kamenasu.blogspot.com/2015/04/go-testing-quick.html)

# 関連
- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018)
- [Goのtestingを理解する in 2018 - Examples編 #go](/2018/08/30/go-testing2018-examples/)
- [Goのtestingを理解する in 2018 - iotestサブパッケージ編 #go](/2018/09/09/go-testing2018-iotest/)

