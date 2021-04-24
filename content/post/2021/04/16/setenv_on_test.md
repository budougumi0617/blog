+++
title= "[Go]環境変数を使ったテストを書く"
date= 2021-04-16T03:38:05+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","gotips","test","env"]
keywords = ["test","env","Go","golang"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goで環境変数を使ってテストを行なうときに便利な関数を作ってみる。

<!--more-->

# TL;DR
- テストケースで特定の環境変数を使いたい
- テストケース終了後は元の環境変数の状態に戻しておきたい
- `os.LookupEnv` と `os.Setenv` を駆使する
    - https://golang.org/pkg/os/#LookupEnv
    - https://golang.org/pkg/os/#Setenv
- Go1.17からは`T.Setenv`が入るので、それを使えばよい
    - https://github.com/golang/go/issues/41260
    - https://github.com/golang/go/blob/ab02cbd29f9b9c76d8f7af0d625ac56fcf8d4e75/src/testing/testing.go#L967-L988

現時点のサンプルコードは以下の通り。

```go
// 環境変数名をキー、環境変数値をバリューとするマップ
func setup(t *testing.T, envs map[string]string) {
  for k, v := range envs {
    prev, ok := os.LookupEnv(k)
    if err := os.Setenv(k, v); err != nil {
      t.Fatalf("cannot set environment key: %q", k)
    }
    k := k // 束縛しておく
    if ok {
      t.Cleanup(func() { _ = os.Setenv(k, prev) })
    } else {
      t.Cleanup(func() { _ = os.Unsetenv(k) })
    }
  }
}
```

# テストケースの中で環境変数を利用したい
なるべく環境変数への依存はおさえるべきだ。とは言え環境変数を使わずにアプリケーションを書くことはほとんどない。
環境変数の利用を含んだテストコードを書いて、そのテストコードの中でだけ環境変数を定義しておきたい。  
また、テストケースが完了したら他のテストケースの邪魔にならないように環境変数をもとに戻したい。


# `os.Getenv`と`os.LookupEnv`
ざっと標準パッケージを見ると環境変数を取得する関数は`os.Getenv`だ。
しかし、`os.Getenv`関数は「未定義の環境変数」からも空文字を取得してしまう。

- https://golang.org/pkg/os/#Getenv

> To distinguish between an empty value and an unset value, use LookupEnv.

よって、「未定義の環境変数」と「空文字が設定された環境変数」を見分けるには`os.LookupEnv`関数を利用する。  
`os.LookupEnv`関数が事前に定義された値を見つければ`t.Cleanup`メソッドでテスト終了後に元の値をセットする。  
そうでなければ`os.UnsetEnv`関数で環境変数の設定を削除すればよい。



# テストコードで環境変数を使いこなすスニペット
`defer`を使うようなこともあったが、Go1.14からは`t.Cleanup`関数を利用すればスッキリ書ける。  
スニペット利用者が正しく後処理関数の実行を行わず環境変数が定義されっぱなしになったという心配もない。  
注意点としては当然ながら`t.Parallel()`関数との併用はできない。

```go
環境変数名をキー、環境変数値をバリューとするマップ
func setup(t *testing.T, envs map[string]string) {
  for k, v := range envs {
    prev, ok := os.LookupEnv(k)
    if err := os.Setenv(k, v); err != nil {
      t.Fatalf("cannot set environment key: %q", k)
    }
    k := k // 束縛しておく
    if ok {
      t.Cleanup(func() { _ = os.Setenv(k, prev) })
    } else {
      t.Cleanup(func() { _ = os.Unsetenv(k) })
    }
  }
}
```

# Go1.17以降は標準パッケージに`T.Setenv`関数が追加される。
ここまで書いたが、このような仕組みはGo1.17で`testing`パッケージ提供される予定だ。

- https://github.com/golang/go/issues/41260
- https://github.com/golang/go/blob/ab02cbd29f9b9c76d8f7af0d625ac56fcf8d4e75/src/testing/testing.go#L1097-L1110


```go
// Setenv calls os.Setenv(key, value) and uses Cleanup to
// restore the environment variable to its original value
// after the test.
//
// This cannot be used in parallel tests.
func (t *T) Setenv(key, value string)
```

リストア処理もやってくれるので、Go1.17以降はこのように書き直せる。

```go
環境変数名をキー、環境変数値をバリューとするマップ
func setup(t *testing.T, envs map[string]string) {
  for k, v := range envs {
    t.Setenv(k, v)
  }
}
```

# 終わりに
さっと書こうとしたが、実は`k := k`を書かないまま動かして少しハマっていた。  
ちゃんとテストを書くのは大事だ。