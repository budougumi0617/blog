+++
title= "[Go]HTTPレスポンスボディの内容をログに残したい"
date= 2020-06-21T19:13:24+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","gotips","net/http"]
keywords = ["Go","Golang","Go言語","net/http","logging"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

GoでHTTPサーバを作成するとき、アクセスログを収集する文脈でレスポンスを記録したくなるだろう。
Middlewareを作成して、HTTPレスポンスをLoggerに出力するやり方をまとめる。

<!--more-->

# TL;DR
- `http.ResponseWriter`に書き込んだ内容をログ出力したい
    - `Read`メソッドは生えていない
- `http.ResponseWriter`はインターフェイスなので挿げ替えができる。
    - `io.MultiWriter`を使って出力を横流しするラッパーを作成する
- レスポンスをloggerに出力するHTTP Middlewareのサンプルを作った。

サンプルコードの主旨は以下。

```go
type rwWrapper struct {
	rw http.ResponseWriter
	mw io.Writer
}

func NewRwWrapper(rw http.ResponseWriter, buf io.Writer) *rwWrapper {
	return &rwWrapper{
		rw: rw,
		mw: io.MultiWriter(rw, buf),
	}
}

func (r *rwWrapper) Header() http.Header {
	return r.rw.Header()
}

func (r *rwWrapper) Write(i []byte) (int, error) {
	return r.mw.Write(i)
}

func (r *rwWrapper) WriteHeader(statusCode int) {
	r.rw.WriteHeader(statusCode)
}

func NewLogger(l *log.Logger) func(http.Handler) http.Handler {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			buf := &bytes.Buffer{}
			rww := NewRwWrapper(w, buf)
			next.ServeHTTP(rww, r)
			l.Printf("%s", buf)
		})
	}
}
```

# レスポンスボディをログに残したい
HTTPサーバを作るとき、リクエストやレスポンスをログとして記録したくなるときがある。

- Pattern: Audit logging
    - https://microservices.io/patterns/observability/audit-logging.html

Goではこのようなとき、HTTP Middlewareを作成することが多い。
リクエストに関しては前処理できるのだが、レスポンスボディをログに残すには一工夫が必要になる。
というのも、Goの`http.Handler`のシグネチャに使われている`http.ResponseWriter`インターフェイスは読み取りに関するメソッドを持っていないからだ。

ただ、`http.ResponseWriter`はインターフェイスなので、ラッパーなどの作成も容易だ。
`http.ResponseWriter`をラップしてロガーにレスポンスボディを記録するHTTP Middlewareを作成してみる。

# http.ResponseWriterを満たすio.MultiWriterを使った構造体を作る
ここでは`io.MultiWriter`関数を使う。

- https://golang.org/pkg/io/#MultiWriter

```go
func MultiWriter(writers ...Writer) Writer
```

`io.MultiWriter`関数の戻り値として得られる`io.Writer`オブジェクトは引数の`writers`の`io.Writer`オブジェクトに同じ内容を書き込む。
これを使ってレスポンスとロガー（で記録するバッファ）に同じ内容を書き込むようなラッパー構造体を作成する。


```go
// http.ResponseWriterインターフェースを満たすためにHeaderとWriteHeaderメソッドも実装する
type rwWrapper struct {
	rw http.ResponseWriter
	mw io.Writer
}

func NewRwWrapper(rw http.ResponseWriter, buf io.Writer) *rwWrapper {
	return &rwWrapper{
		rw: rw,
		mw: io.MultiWriter(rw, buf),
	}
}

func (r *rwWrapper) Write(i []byte) (int, error) {
	return r.mw.Write(i)
}
```

`rwWrapper`構造体は`NewRwWrapper`関数で受け取った`http.ResponseWriter`の薄いラッパー構造体だ。
レスポンスボディへの書き込みが行われるとき、`io.Writer`を使って`buf`オブジェクトにその内容を横流しする。


あとは実際のハンドラーにわたす`http.ResponseWriter`オブジェクトをすげ替えるMiddlewareを作成する。
実際のハンドラーの処理がおわったあとにバッファからレスポンスを読み出せばよい。

```go
// ロガーをDIしてmiddlewareを返す
func NewLogger(l *log.Logger) func(http.Handler) http.Handler {
    // nextハンドラーのレスポンスをロガーに出力するmiddleware
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			buf := &bytes.Buffer{}
			rww := NewRwWrapper(w, buf)
			next.ServeHTTP(rww, r)
			l.Printf("%s", buf)
		})
	}
}
```


# 動作確認（テストコード）
作成したHTTP Middlewareのテストコードを書いてみる。
クライアントとMiddelewareを呼び出すサーバを作り、クライアントが受信したデータとロガーに流し込まれたデータを比較する。

- https://play.golang.org/p/2JoamybGQzn

```go
package main

import (
	"bytes"
	"context"
	"fmt"
	"io"
	"io/ioutil"
	"log"
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"
	"time"
)

type rwWrapper struct {
	rw http.ResponseWriter
	mw io.Writer
}

func NewRwWrapper(rw http.ResponseWriter, buf io.Writer) *rwWrapper {
	return &rwWrapper{
		rw: rw,
		mw: io.MultiWriter(rw, buf),
	}
}

func (r *rwWrapper) Header() http.Header {
	return r.rw.Header()
}

func (r *rwWrapper) Write(i []byte) (int, error) {
	return r.mw.Write(i)
}

func (r *rwWrapper) WriteHeader(statusCode int) {
	r.rw.WriteHeader(statusCode)
}

func NewLogger(l *log.Logger) func(http.Handler) http.Handler {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			buf := &bytes.Buffer{}
			rww := NewRwWrapper(w, buf)
			next.ServeHTTP(rww, r)
			l.Printf("%s", buf)
		})
	}
}

func TestLogger(t *testing.T) {
	h := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		_, _ = fmt.Fprint(w, "Hello, client")
	})
	buf := &bytes.Buffer{}
	l := log.New(buf, "", 0)
	ts := httptest.NewServer(NewLogger(l)(h))
	t.Cleanup(ts.Close)

	cli := &http.Client{
		Timeout: 2 * time.Second,
	}
	req, err := http.NewRequestWithContext(context.TODO(), "GET", ts.URL, nil)
	if err != nil {
		t.Fatal(err)
	}

	resp, err := cli.Do(req)
	if err != nil {
		t.Fatal(err)
	}

	// compare logging data and client received value.
	want, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		t.Fatal(err)
	}
	_ = resp.Body.Close()
	// \nが混ざっているので、完全一致にはならない
	if !strings.Contains(buf.String(), string(want)) {
		t.Errorf("want %q, but %q", want, buf)
	}
}
```

実際に動かすとロガーにレスポンスボディが流れていることを確認できた。

# 終わりに
実際に使うときは標準pkgのロガーではなくOSSのロガーを使うだろうが大体同じ構造になるはずだ。
ただし、レスポンスにはセンシティブな情報が含まれる場合もある。必要に応じてマスキングなどの作り込みが必要だろう。

# 参考
- https://play.golang.org/p/dSpnSfQjHi5
- https://microservices.io/patterns/observability/audit-logging.html
- https://golang.org/pkg/net/http/#ResponseWriter
- https://golang.org/pkg/io/#MultiWriter
