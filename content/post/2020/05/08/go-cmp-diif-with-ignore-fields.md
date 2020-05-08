+++
title= "[Go] 一部のフィールドを無視して構造体を比較したいときはgo-cmpを使う"
date= 2020-05-08T08:31:18+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["test","golang"]
keywords = ["Go","Go言語","golang","test","go-cmp"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goでテストを書くとき、期待値として構造体を比較したいときは多々ある。  
時刻情報など、構造体の一部のフィールドの値だけ無視して構造体オブジェクトを比較する方法をまとめた。

<!--more-->

# TL;DR
- 構造体を比較するとき`time.Time`や`ID`フィールドは無視して比較したい
- go-cmpは構造体の差分を表示することができるライブラリ
    - https://github.com/google/go-cmp
- `IgnoreFields`を使うと、任意のフィールドを無視して比較することができる
    - https://godoc.org/github.com/google/go-cmp/cmp/cmpopts#IgnoreFields

```go
import (
  "time"
  "testing"

  "github.com/google/go-cmp/cmp"
  "github.com/google/go-cmp/cmp/cmpopts"
)

type Foo strcut{
    Name      string
    Timestamp time.Time
}

func TestLogger(t *testing.T) {
  var got Foo
  // Arrange, Action...

  // Assert。TimeStampフィールドは無視して比較する
  if d := cmp.Diff(got, want, cmpopts.IgnoreFields(got, "TimeStamp")); len(d) != 0 {
    t.Errorf("differs: (-got +want)\n%s", d)
  }
}
```

# 構造体の一部を無視して比較したい
Goのテストを書いていると、ある構造体が期待通りの値になったか確認したくなるだろう。
その中で一部の値は無視して期待値と比較をしたくなるときがある。

- DB保存時に自動裁判されるIDは無視して比較したい
- `CreatedAt`のような時刻情報は無視して比較したい

たとえば、次のような構造体をDBから取得するテストを書く場合、`ID`、`CreatedAt`、`ModifiedAt`のフィールドを無視したいだろう。
大抵の場合、DBで自動採番されたIDやデータの作成時刻などはテストケースで検証すべきことではないからだ。

```go
type SimpleObject struct {
  ID         int       `json:"id" db:"id"`
  Name       string    `json:"name" db:"name"`
  Address    string    `json:"address" db:"address"`
  CreatedAt  time.Time `json:"created_at" db:"created_at"`
  ModifiedAt time.Time `json:"modified_at" db:"modified_at"`
}
```


愚直に書いてしまうと、次のようなコードになってしまう。

```go
func TestSimple(t *testing.T) {
  var got SimpleObject
  var want SimpleObject
  // Arrange, Action...

  // Assert ID、CreatedAt、ModifiedAt以外のフィールドの値を検証していく。
  if got.Name != want.Name {
    t.Errorf("want %v, but got %v", want.Name, got.Name)
  }
  if got.Address != want.Address {
    t.Errorf("want %v, but got %v", want.Address, got.Address)
  }
}
```

複雑な構造体ほど比較すべき項目が増えていってしまう。
このようなとき、`go-cmp`ライブラリを使うとすっきりした検証処理を記載できる。

# go-cmpを使うと任意のフィールドを除外して等価性を評価できる
`go-cmp`は複雑な構造体オブジェクト同士の等価性や差分を取得するためのライブラリだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/google/go-cmp" data-iframely-url="//cdn.iframe.ly/BQ0axoF"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

次のように`Diff`関数で構造体を比較を実行する。

```go
if diff := cmp.Diff(want, got); diff != "" {
  t.Errorf("MakeGatewayInfo() mismatch (-want +got):\n%s", diff)
}
```

構造体の値に差分があった場合、次のような文字列出力が得られる。

```bash
MakeGatewayInfo() mismatch (-want +got):
  cmp_test.Gateway{
    SSID:      "CoffeeShopWiFi",
-   IPAddress: "192.168.0.2",
+   IPAddress: "192.168.0.1",
    NetMask:   net.IPMask{0xff, 0xff, 0x00, 0x00},
    Clients: []cmp_test.Client{
    ...
```

## IgnoreFieldsオプションを使って特定のフィールドを無視する
`go-cmp`は単純に構造体を比較するだけでなく、オプションを使ってさまざまな比較をすることができる。
そのなかの1つが`IgnoreFields`オプションだ。  
`IgnoreFields`を渡して`Diff`関数を実行すると、`IgnoreFields`で指定されたフィールドの値を無視して比較を行なうことができる。

使い方は簡単で、`Diff`関数実行時に無視して欲しいフィールド名を渡すだけだ。
オプションなしで次のようなテストコードを書いてみる。

```go
// https://play.golang.org/p/m0K3Ub5LSPY
package main

import (
  "testing"

  "github.com/google/go-cmp/cmp"
)

type SimpleObject struct {
  ID        int
  Name      string    `json:"name" db:"name"`
  Address   string    `json:"address" db:"address"`
}

func TestSimple(t *testing.T) {
  got := &SimpleObject{
    ID:   100,
    Name: "same name",
  }
  want := &SimpleObject{
    ID:   200,
    Name: "same name",
  }

  if d := cmp.Diff(got, want); len(d) != 0 {
    t.Errorf("differs: (-got +want)\n%s", d)
  }
}
```

この場合、`ID`が異なるので当然結果はエラーになる。

```diff
=== RUN   TestSimple
    TestSimple: prog.go:29: differs: (-got +want)
          &main.SimpleObject{
        -   ID:        100,
        +   ID:        200,
            Name:      "same name",
            Address:   "",
            CreatedAt: s"0001-01-01 00:00:00 +0000 UTC",
          }
--- FAIL: TestSimple (0.00s)
FAIL
```

ここで、`IgnoreFields`を使って`ID`フィールドを無視する設定を書く。
こうすると、`ID`フィールド以外は差分がないので、テストがパスするようになる。

```go
// https://play.golang.org/p/h1uFWBxUqxc
if d := cmp.Diff(got, want, cmpopts.IgnoreFields(*got, "ID")); len(d) != 0 {
  t.Errorf("differs: (-got +want)\n%s", d)
}
```

また、`IgnoreFields`はネストした構造体のフィールドにも適用できる。

```go
type Foo struct{
  Simple *SimpleObject
}

type SimpleObject struct {
  ID      int
  Name    string `json:"name" db:"name"`
  Address string `json:"address" db:"address"`
}
```

`Foo`構造体にネストしている`*SimpleObject`の`ID`フィールドを無視して`Foo`オブジェクトを比較したいときは次のようになる。

```go
cmpopts.IgnoreFields(Foo{}, "Simple.ID")
```

# 終わりに
`go-cmp`ライブラリ自体は昔から使っていたのだが、先日`IgnoreFields`オプションをはじめて知った。  
ライブラリのサブパッケージまでちゃんと確認していなかったのが、原因だ。  
思わぬ使い方があったり、または誤用してしまう可能性もあるため、ライブラリのGoDocはひととおり読んで使っていきたい。


# 参考
- https://github.com/google/go-cmp
- [Goで時刻が絡む単体テストをどうするか考える](https://daisuzu.hatenablog.com/entry/2018/05/05/010510)
