+++
title= "[Go] 埋め込みフィールドを使ったラッパー構造体定義"
date= 2020-11-01T17:59:43+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","gotips"]
keywords = ["Go","wrapper","埋め込み"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goである構造体（例:サードパーティのクライアント）のラッパーを書くときは埋込みフィールドを使うといいよという話。

<!--more-->

# TL;DR
- 埋め込みフィールド（embedded field）を使うとラッパー構造体を簡単に実装できる
    - https://golang.org/ref/spec#Struct_types
- インターフェイスを使った埋め込みフィールドも定義可能
    - https://play.golang.org/p/FWiWDdozeIl
- 初期化を忘れると初期化関数を作っておく
    - https://golang.org/doc/effective_go.html#composite_literals

# ある構造体をラップしたい
デコレータパターンのようにある構造体のラッパーを作りたいときがある。  
たとえばサードパーティのクライアントをラップして一部のメソッドの動きだけ少し変えたいときだ。

今回は次のような構造体をベースに考える。  
`OriginClient`構造体は`Client`インターフェイスを満たす構造体だ。

```go
// 満たすべきインターフェイス
type Client interface {
    PrintValue()
    ShowInt(int) error
    OtherMethod() error
}

// interfaceを満たしているか。
var _ Client = &OriginClient{}

type OriginClient struct{
    Value int
}

func (o *OriginClient) PrintValue() { fmt.Printf("show data %d\n", o.Value)}

func (o *OriOriginClientgin) ShowInt(i int) error {
    if i < 0 {
        return fmt.Errorf("not support negative value %d", i)
    }
    fmt.Printf("origin: %d\n", i)
    return nil
}
func (o *OriginClient) OtherMethod() error { return nil }
```

たとえば、この構造体に次のような機能を追加したラッパーを作りたいとする。  

- **`ShowInt`メソッドでエラーが発生したときはログ出力する**
- 他のメソッドの処理はそのままでよい

何も考えずに実装するとこうなる。

- https://play.golang.org/p/a94bS26PX8A

```go
var _ Client = &MyClient{}
// ShowIntメソッドだけラップしているラッパー
type MyClient struct {
	oc *OriginClient
}

func (m *MyClient) PrintValue() { m.oc.PrintValue() }

// ラップメソッド
func (m *MyClient) ShowInt(i int) error {
	if err := m.oc.ShowInt(i); err != nil {
		log.Printf("errorが発生したときはログ出力する: %v\n", err)
		return err
	}
	return nil
}

func (m *MyClient) OtherMethod() error { return m.oc.OtherMethod() }

func main() {
	mc := &MyClient{oc: &OriginClient{Value: 100}}
	mc.ShowInt(10)
}
```

これくらいのメソッド数ならば良いが、各メソッドごとにラップメソッドを定義するのはめんどくさい。

# 埋込みフィールドを使うと定義を省略することができる
このような場合、埋め込みフィールドを使うとメソッド定義を省略することができる。

- https://play.golang.org/p/vzngwQ2pqYG

```go
var _ Client = &MyClient{}
// ShowIntメソッドだけラップしているラッパー
type MyClient struct {
	*OriginClient
}

// ラップメソッド
func (m *MyClient) ShowInt(i int) error {
	if err := m.OriginClient.ShowInt(i); err != nil {
		log.Printf("errorが発生したときはログ出力する: %v\n", err)
		return err
	}
	return nil
}

func main() {
	mc := &MyClient{OriginClient: &OriginClient{Value: 100}}
	mc.ShowInt(10)
}
```

大きく変わっている定義はここ。
```go
type MyClient struct {
	*OriginClient // フィールドではない。
}
```

埋め込みフィールドを使うと、`MyClient`構造体で明示的に実装していないメソッドに関しては、`OriginClient`のメソッドが直接呼ばれる。
そして`MyClient`はそれらのメソッドを実装していることになるので、`OriginClient`構造体が満たすインターフェイスと同じインターフェイスを満たすことができる。

- https://golang.org/doc/effective_go.html#composite_literals

> - If S contains an embedded field T, the method sets of S and *S both include promoted methods with receiver T. The method set of *S also includes promoted methods with receiver *T.
> - If S contains an embedded field *T, the method sets of S and *S both include promoted methods with receiver T or *T.

構造体フィールドとして宣言しているわけではないが、初期化するときは次のように代入できる。

```go
mc := &MyClient{OriginClient: &OriginClient{Value: 100}}
```
また、埋め込んだ構造体のメソッドを明示的に呼びたいときは次のように書ける。
```go
func (m *MyClient) ShowInt(i int) error {
	m.OriginClient.ShowInt(i)
	// do anything...
}
```

# unexportedな構造体のラッパーを書きたいときは？
ライブラリによっては構造体自体が公開されていないこともある。
たとえば次のようなつくりのライブラリもあるだろう。

```go
package thirdparty

type Client interface {
	PrintValue()
	ShowInt(int) error
	OtherMethod() error
}

func NewClient(value int) Client {
	return &originClient{value} // このoriginClientのラッパーを書きたい
}
```

このような場合もインターフェイスを埋め込むことでラッパーを作成することが可能だ。

- https://play.golang.org/p/VhRPUPDygm9

```go
package main

var _ thirdparty.Client = &MyClient{}

// ShowIntメソッドだけラップしているラッパー
type MyClient struct {
	thirdparty.Client
}

// ラップメソッド
func (m *MyClient) ShowInt(i int) error {
	if err := m.Client.ShowInt(i); err != nil {
		log.Printf("errorが発生したときはログ出力する: %v\n", err)
		return err
	}
	return nil
}

func main() {
	mc := &MyClient{Client: thirdparty.NewClient(100)}
	mc.ShowInt(10)
}
```

# ラッパーの初期化に注意すること
以上のように埋め込みを使うことで簡単にラッパーメソッドを作成することができる。
しかし、ポインタやインターフェイスを埋め込んだ場合は明示的に初期化をしないと実行時にパニックが発生する。

```go
// Bメソッドだけラップしているラッパー
type MyClient struct {
	thirdparty.Client // インターフェイス
}

func main() {
    mc := &MyClient{}
    // panic: runtime error: invalid memory address or nil pointer dereference
	mc.ShowInt(10)
}
```

`NewClient`関数などの初期化関数を用意しておくとよいだろう。

https://golang.org/doc/effective_go.html#composite_literals

# 終わりに
知っている人はけっこう多いテクニックだが先日「ラッパー書くときどうすればいいの？」と聞かれたのでブログにしておいた。


# 参考
- https://golang.org/ref/spec#Struct_types
- https://golang.org/doc/effective_go.html#composite_literals
- https://play.golang.org/p/vzngwQ2pqYG
- https://play.golang.org/p/a94bS26PX8A
- https://play.golang.org/p/VhRPUPDygm9