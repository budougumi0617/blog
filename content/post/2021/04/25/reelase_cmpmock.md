+++
title= "gomockでモックメソッドの引数をいい感じに設定できるcmpmockを作った"
date= 2021-04-25T08:04:52+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["test","golang","gomock","oss"]
keywords = ["Go","golang","test","モック","oss"]
twitterImage = "twittercard.png"
+++


`gomock`（`github.com/golang/mock`）のモックメソッドの引数をいい感じに設定できるカスタムマッチャーを作った。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/cmpmock" data-iframely-url="//cdn.iframe.ly/TTjYmqy?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
<br>

```go

mrepo.EXPECT().Save(ctx, cmpmock.DiffEq(wantUser)).Return(nil)
```

<!--more-->

# TL;DR
- Goのテストでモックを使いたいときのデファクトスタンダードはgomock
    - https://github.com/golang/mock
- モックのメソッド引数に期待値を指定するとき諦めたくなる時がある
    - `gomock.Any()` でごまかして終わりそうになる
- gomock用にカスタムマッチャーを作った
    - https://github.com/budougumi0617/cmpmock
    - `cmpmock.DiffEq`
    - 期待値を異なっていたときの差分が見やすい
    - `google/go-cmp/cmp#Option` で挙動を変更できる
- 内部で`google/go-cmp` を使っている
    - https://github.com/google/go-cmp/


使い方はこんな感じ。

```go
type UserRepo interface {
  Save(context.Context, *User) error
}

wantUser := &User{}
mrepo := mock.NewMockUserRepo(ctrl)
mrepo.EXPECT().Save(ctx, cmpmock.DiffEq(wantUser)).Return(nil)
```

# Goのテストでモックを使う
- https://github.com/golang/mock

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/mock" data-iframely-url="//cdn.iframe.ly/RivSfnz?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Goでモックをテストに使いたいとき、デファクトスタンダードとして使われているのが`gomock`だろう。
本記事は`gomock`用にカスタムマッチャーを作った話なのだが、`gomock`自体の説明は省略する。
使ったことがない方はpkg.go.devを見てみると雰囲気がわかるだろう。

https://pkg.go.dev/github.com/golang/mock@v1.5.0/gomock


## モックのメソッド引数に期待値を指定するとき諦めたくなる時がある
`gomock`は柔軟に挙動を変えたり呼び出し回数や呼び出し順まで検証することができる。  
ただ、モックの引数にそこそこでかい構造体を指定したり、時刻情報を用いているとうまく期待値を設置できないときがないだろうか？
そういうときは`gomock.Any()`関数を引数に指定したりして、モックの設定を妥協したりすることがあった。
毎回`DoAndReturn`などで構造体をパースする処理をテストコードに書くのもめんどくさい。

- [[Go][gomcok]引数によって挙動を変えるモックメソッドを定義する #golangjp](/2019/03/10/define-gomock-method-by-doandreturn/) 

また、指定できていたとしても期待値と異なる値がモックメソッドに指定されたとき「どこか期待と異なるのか」が非常にわかりにくい。

たとえば、モックメソッドの引数が次の`User`構造体の場合をみてみる、

```go
type User struct {
  Name, Address string
  CreateAt      time.Time
}
```

期待値と異なった場合、テスト実行時に以下が出力されるのだが、どこに差異があるのかわかるだろうか？

```bash
expected call at /Users/budougumi0617/go/src/github.com/budougumi0617/cmpmock/_example/repo_test.go:26 doesn't match the argument at index 1.
Got: &{John Due Tokyo 2021-04-23 02:46:58.145696 +0900 JST m=+0.000595005}
Want: is equal to &{John Due Tokyo 2021-04-23 02:46:48.145646 +0900 JST m=-9.999455563}
```


## gomock用にカスタムマッチャーを作った
- https://github.com/budougumi0617/cmpmock

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/cmpmock" data-iframely-url="//cdn.iframe.ly/TTjYmqy?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`gomock`のモックメソッドの引数は`gomock#Matcher`インターフェイスを満たす構造体を定義すれば自分でカスタムマッチャーを用意することができる。
`gomock.Any()`関数も定義済み`gomock#Matcher`の一種だ。

- https://pkg.go.dev/github.com/golang/mock@v1.5.0/gomock#Matcher


使い方は次のように使う。「いい感じに比較してほしい」モックメソッドの期待値を指定するときに`cmpmock.DiffEq`メソッドでラップして指定すればよい。

```go
type UserRepo interface {
  Save(context.Context, *User) error
}

wantUser := &User{}
mrepo := mock.NewMockUserRepo(ctrl)
mrepo.EXPECT().Save(ctx, cmpmock.DiffEq(wantUser)).Return(nil)
```

期待結果と異なると、以下のような出力が得られる。

```
expected call at /Users/budougumi0617/go/src/github.com/budougumi0617/cmpmock/_example/repo_test.go:27 doesn't match the argument at index 1.
Got: &{John Due Tokyo 2021-04-23 02:46:33.290458 +0900 JST m=+0.001035665}
Want: diff(-got +want) is   &_example.User{
  Name:     "John Due",
  Address:  "Tokyo",
-   CreateAt: s"2021-04-23 02:46:33.290458 +0900 JST m=+0.001035665",
+   CreateAt: s"2021-04-23 02:46:23.290383 +0900 JST m=-9.999039004",
}
```

前述の出力結果と比較してだいぶわかりやすい。

# 内部で`google/go-cmp`を使っているだけ

- https://github.com/google/go-cmp/

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/google/go-cmp" data-iframely-url="//cdn.iframe.ly/Q4XPfBv?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

出力結果を見て気づく方はすぐわかるだろうが、`DiffEq`メソッドは内部で`go-cmp`を使って期待値と入力値を比較しているだけだ。  
なので正直OSSというより、既存のOSSを組み合わせただけのスニペットである。

## 挙動はgo-cmpのcmpoptsなどで変更できる

`DiffEq`関数はシグネチャとして`cmp.Option`を受け取るので、`go-cmp`を使いなれているひとならば柔軟に比較方法を変更できる。

```go
func DiffEq(v interface{}, opts ...cmp.Option) gomock.Matcher
```

何も指定しない場合は`time.Time`の差が1秒未満だった場合無視するようにしている。  
テストでそこまで厳密な時刻の比較は必要にないと思っている（というより、これが原因で比較を諦めることが大半…）。

```go
func DiffEq(v interface{}, opts ...cmp.Option) gomock.Matcher {
  var lopts cmp.Options
  if len(opts) == 0 {
    lopts = append(lopts, cmpopts.EquateApproxTime(1*time.Second))
  } else {
    lopts = append(lopts, opts...)
  }
  return &diffMatcher{want: v, opts: lopts}
}
```

# 終わりに
モックの設定に時間を書けるのは非常にストレスなのでいい感じにモック引数を設定できる`DiffEq`関数を作った。  
既存のOSSを組み合わせただけだが、コスパよく便利なマッチャーを作れた気がする。  
これでもっとテストが書けるぞ！！