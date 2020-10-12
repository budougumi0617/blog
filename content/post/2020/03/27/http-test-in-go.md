+++
title= "Goでサーバを立ち上げてE2Eテストを実施するCI用のテストコードを書く"
date= 2020-03-27T20:00:22+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","test","e2e","gotips"]
keywords = ["Go","Golang","Go言語","test","E2Eテスト"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


GoでCIで動かせるE2Eテストコードを書くための下調べをしたのでメモしておく。

<!--more-->

# TL;DR
- CIで動かせるE2Eテストとして、`ListenAndServe`しているサーバに対してHTTP通信するテストコードを書きたくなった。
- `ListenAndServe`は通信の開始待ちができないので、Flakyなテストになってしまう。
- `net.Listen`して取得した`net.Listener`を使うことで、待機無しでテスト対象のサーバと通信できる。

# CIで実行できるE2EレベルのテストをGoのテストコードとして書きたい

[柴田](https://twitter.com/yoshiki_shibata/)さんや[@t_wada](https://twitter.com/t_wada)さんの話を読んで、もっとE2Eテストに近いHTTPテストについてちゃんと考えたいなと思い始めた。

- マイクロサービスの開発とテストファースト/テスト駆動開発 | GDG Dev Fest Tokyo 2019
    - https://www.slideshare.net/yoshikishibata/gdg-dev-fest-tokyo-2019

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">モック多用の自作自演テストやレイヤ間のテスト範囲重複を好まないので（個人の意見）、テストを内側と外側に寄せます。Railsの場合はRequest spec(HTTPレベルのテスト、E2Eより安定していてかつ粗粒度) とModel spec（ドメインロジックのテストを厚く書く）を基本的に書き、Controllerは例外系のみ。</p>&mdash; Takuto Wada (@t_wada) <a href="https://twitter.com/t_wada/status/1237200607045251073?ref_src=twsrc%5Etfw">March 10, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

今まで、`httptest.ResponseRecorder`などを利用したハンドラーレベルのテストは実施していた。  

```go
handler := func(w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, "<html><body>Hello World!</body></html>")
}

// Arrange
req := httptest.NewRequest("GET", "http://example.com/foo", nil)
w := httptest.NewRecorder()

// Act
handler(w, req)

// Assert
```
が、サーバを起動してそのサーバに対してHTTPリクエストを飛ばして挙動を確認するところまでやりたいなと思った。  
もう少し具体的に書くと、自身が作っているWebアプリに対して、次のようなテストシナリオをGoのテストコードとして書きたい。

```go
func TestAPIServer(t *testing.T) {
    // テスト対象のWebアプリサーバが依存する各種サービスのフェイクサーバを起動する（このブログ記事では触れない）
    // テスト対象のWebアプリサーバを起動する
    // テスト対象のサーバに対してHTTPリクエストを送信する
    // レスポンス内容を検証する
    // 各種サーバを適切に終了する
}
```
# HTTPテストを書くときの問題
上記のようなテストコードを書こうとしたとき、テスト対象のWebアプリサーバの起動部分に`ListenAndServe`メソッドを使っていると問題が発生する。

- https://play.golang.org/p/K68LHrm_IE8

```go
func TestStartServer(t *testing.T) {
    want := "Hello, world!\n"
    mux := http.NewServeMux()
    mux.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
        io.WriteString(w, want)
    })

    s := &http.Server{
        Addr:    ":8080",
        Handler: mux,
    }

    go func() {
        t.Fatal(s.ListenAndServe())
    }()
    time.Sleep(0 * time.Second) // 1秒待機すれば成功する
    res, err := http.Get("http://127.0.0.1:8080/")
    if err != nil {
        // Get http://127.0.0.1:8080/: dial tcp 127.0.0.1:8080: connect: Connection refused
        t.Fatal(err)
    }
    b, err := ioutil.ReadAll(res.Body)
    res.Body.Close()
    if err != nil {
        t.Fatal(err)
    }
    if string(b) != want {
        t.Fatalf("want %q, but %q", want, b)
    }
}
```
`http.Server#ListenAndServe`メソッド（`http.ListenAndServe`関数）は呼び出すと通信終了まで完了しない。  
そのためテストコード中で、`ListenAndServe`メソッドを実行してテストを開始するには、別goroutineで実行する必要がある。  
そうなると、当然サーバが起動するまでテストコードから通信するのを待つ必要がある。  
数秒待てばサーバは起動しているが、タイミングを合わせるためにテストの実行時間をむやみに伸ばしたくはない。  
かと言って、ギリギリの時間まで待機時間を削ると、Flakyなテストになるのでそれも良くない。

- 【用語解説】John Micco氏の言う"Flaky"なテストとは？
    - https://jasst-tokyo.hatenablog.jp/entry/2018/02/22/214551

# net.Listen関数を使って、待ち無しでListen状態にしておく
この問題を解決するには、`net.Listen`を明示的に宣言して、`http/Server.Serve`メソッド（`http.Serve`関数）からサーバを起動すればよい。  
テスト対象のサーバを起動して、待ち無しでリクエストを送信、レスポンスを検証するエンドポイントテストが以下だ。

- https://play.golang.org/p/hg-lNU7JFIP

```go
package main

import (
    "context"
    "io"
    "io/ioutil"
    "log"
    "net"
    "net/http"
    "testing"
)

func buildServer(body string) *http.Server {
    mux := http.NewServeMux()
    mux.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
        io.WriteString(w, body)
    })

    return &http.Server{
        Handler: mux,
    }
}

func TestStartServer(t *testing.T) {
    want := "Hello, world!\n"

    // Arrange
    srv := buildServer(want)

    // 動的にポートを選択するので並行テストが可能。
    l, err := net.Listen("tcp", ":0")
    if err != nil {
        log.Fatal(err)
    }

    idleConnsClosed := make(chan struct{})
    go func() {
        if err := srv.Serve(l); err != http.ErrServerClosed {
            t.Fatalf("HTTP server ListenAndServe: %v", err)
        }
        // サーバが終了したことを通知。
        close(idleConnsClosed)
    }()

    // Act
    res, err := http.Get("http://" + l.Addr().String())
    if err != nil {
        t.Fatal(err)
    }
    b, err := ioutil.ReadAll(res.Body)
    if err != nil {
        t.Fatal(err)
    }
    res.Body.Close()

    // Assert
    if string(b) != want {
        t.Fatalf("want %q, but %q", want, b)
    }

    // Cleanup
    if err := srv.Shutdown(context.Background()); err != nil {
        t.Fatalf("HTTP server Shutdown: %v", err)
    }

    // サーバの終了を確認してからテストコードを終了する。
    <-idleConnsClosed
}
```

`net.Listen`関数は待ちを発生させずに完了し、関数呼び出し終了時点で`LISTEN`が開始される。

- https://golang.org/pkg/net/#Listen

そのため、別goroutineを呼ばずにサーバ用の通信を開始することができる。  
あとは、この`net.Listener`オブジェクトを使ってテスト対象のサーバを別goroutineで起動する。  
こうすると、テストコードは何も待たずに通信を始めることができる。

テストコードの実行が終了したら、`http.Server#Shutdown`メソッドを実行することでグレースフルにサーバを終了することができる。

- https://golang.org/pkg/net/http/#Server.Shutdown

テスト対象のアプリサーバの起動処理を、このような呼び出し方に対応するように設計する必要があるが、これでHTTP通信を利用したテストコードが書けそうだ。

# おわりに
テスト対象のサーバの起動インターフェイスに条件はあるものの、やりたいことができるのは確認した。  
最初どうやってサーバの起動を待機しようかと考えていた。  
`httptest.NewServer`は待機処理をせずとも通信が成功するので、そのコードを参考にした。  
（`httptest.NewServer`をそのまま使えばいいのかもしれないが…）  
`net.Listen`を明示的に始めることで、その時点からTCP通信を待機状態にするのがコツだった。  
このあたりは[Goならわかるシステムプログラミング](https://amzn.to/2wFdLyy)のTCPソケットの章を読み直して復習できた。

途中、Webアプリのユニットテスト書こうとしたのに、システムプログラミングに行き着いた。すべてがつながってる感がある。

今回は省略したが、やりたいテストコードを書くには、テスト対象が依存するサービスのフェイクサーバも起動しないといけない。  
本文中の書きなぐりのコードでは利用が難しいので、起動部分を再利用可能なカタチにまとめて、フェイクサーバを起動したり、複数のテストで利用できるパッケージにしておきたいなと思う。


# 参考文献
- [マイクロサービスの開発とテストファースト/テスト駆動開発 | GDG Dev Fest Tokyo 2019](https://www.slideshare.net/yoshikishibata/gdg-dev-fest-tokyo-2019)
- [net/http | The Go Programing Language](https://golang.org/pkg/net/http/)
- [【用語解説】John Micco氏の言う"Flaky"なテストとは？](https://jasst-tokyo.hatenablog.jp/entry/2018/02/22/214551)
- [net.Listen | The Go Program Language](https://golang.org/pkg/net/#Listen)
- [Goならわかるシステムプログラミング](https://amzn.to/2wFdLyy)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4908686033&linkId=a9a82764bbb3fb3c7346aa7590a81ad8"></iframe>
