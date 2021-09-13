+++
title= "[Go] 前方互換性を保ちながらhttp.DefaultTransportからチューニングしたhttp.Transportをつくる"
date= 2021-09-13T10:00:13+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["net/http","http","golang","client","gotips"]
keywords = ["net/http","transport","roundtripper"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


[@dice_zu](https://twitter.com/dice_zu)さんから`http.DefaultTransport`の正しい（？）コピーのやり方を教えてもらったのでメモしておく。  
結論から言うと`http.DefaultTransport`変数にたいして`net/http#Transport.Clone`メソッドを使うと良い。
これなら新しいGoのバージョンで`http.Transport`に新しいフィールドが追加されても問題ない。

https://pkg.go.dev/net/http#Transport.Clone

<!--more-->


# TL;DR
- `*http.Client`オブジェクトは再利用したほうがよい
    - https://pkg.go.dev/net/http#Client
    - 内部でTCPコネクションのキャッシュを持っているから
- `http.DefaultClient`はタイムアウトの設定がされていないので独自定義するのが一般的
- `http.Transport`オブジェクトも独自定義する
- `http.DefaultTransport`から少しだけ設定値を変更したオブジェクトを作るには`Clone`メソッドを使う。
    - https://pkg.go.dev/net/http#Transport.Clone


サンプルコードは次の通り。

https://play.golang.org/p/niRAgxrIv8V
```go
package main

import (
  "net"
  "net/http"
  "time"
)

func main() {
  t := defaultTransport()
  t.MaxIdleConnsPerHost = 100
  cli := &http.Client{
    Transport: t,
    Timeout:   3 * time.Second,
  }
  _, _ = cli.Get("https://example.com")
}

func defaultTransport() *http.Transport {
  dt := http.DefaultTransport
  if t, ok := dt.(*http.Transport); ok {
    return t.Clone()
  }
  // 何らか悪されてていた時。
  return &http.Transport{
    Proxy: http.ProxyFromEnvironment,
    DialContext: (&net.Dialer{
      Timeout:   30 * time.Second,
      KeepAlive: 30 * time.Second,
    }).DialContext,
    ForceAttemptHTTP2:     true,
    MaxIdleConns:          100,
    MaxIdleConnsPerHost:   100,
    IdleConnTimeout:       90 * time.Second,
    TLSHandshakeTimeout:   10 * time.Second,
    ExpectContinueTimeout: 1 * time.Second,
  }
}
```

# `*http.Client`オブジェクトは再利用したほうがよい
Goで何かしらのHTTPリクエストを送信したいならば、`*http.Client`オブジェクトを使う。
Webアプリケーションのサーバを書いているときでも外部サービスや他のバックエンドサービスへ通信をするために多用する。

- https://pkg.go.dev/net/http#Client

このとき、`*http.Client`オブジェクトはリクエストのたびに生成してはいけない。可能な限り再利用する必要がある。
その理由は`http.Client`の定義に仕様として記載がある通り、`*http.Client`オブジェクトは内部でTCPコネクションのキャッシュを持つからである。

> The Client's Transport typically has internal state (cached TCP connections), so Clients should be reused instead of created as needed. Clients are safe for concurrent use by multiple goroutines.

# `http.DefaultClient`はタイムアウトの設定がされていないので独自定義するのが一般的
他方、`*http.Client`オブジェクトは`net/http`パッケージに宣言済みの`http.DefaultClient`変数が存在する。これを使うことは基本的に避ける。
なぜならば、この`http.DefaultClient`はタイムアウトの設定がされていないためだ。そのため、`*http.Client`オブジェクトを用意するときは次のように宣言したりする。

```go
cli := &http.Client{
    // Transport: 未初期化の場合はhttp.DefaultTransportが呼ばれる。
    Timeout:   3 * time.Second,
}
```

ここで、`*http.Client.Transport`フィールドも独自定義の設定に変えたい時がある。
`*http.Client.Transport`フィールドは未初期化状態だった場合、`http.DefaultTransport`が呼ばれる。

# `*http.Transport`オブジェクトも独自定義する
`http.Transport`型には`MaxIdleConnsPerHost`というフィールドがある。名前の通り、ホストごとのアイドルするコネクションの最大値を制御する設定だ。
未初期化だと`http.DefaultMaxIdleConnsPerHost`が利用される。Go1.17時点で`http.DefaultMaxIdleConnsPerHost`は`2`だ。
マイクロサービス構成のバックエンドサーバなどで`*http.Client`オブジェクトを利用するシーンでは外部サービスや別のマイクロサービスサーバなど接続先ホストがいくつかに限定されることが多いだろう。限定された接続先にスループットの数だけ並列リクエストが飛ぶことを考慮すると`MaxIdleConnsPerHost`を`2`以上に設定したくなる。

```go
transport := &http.Transport{
        DefaultMaxIdleConnsPerHost: 100,
  }
```

しかし上記のような宣言では`MaxIdleConnsPerHost`フィールド以外のフィールドがゼロ値になってしまうため、よくない宣言だ。
ではどのように`*http.Transport`オブジェクトを初期化すればよいのだろうか？一番イージーなのは`http.DefaultTransport`変数の宣言をコピペだろう。
Go1.17時点の`http.DefaultTransport`変数の宣言は次のような設定値で初期化されている。

https://github.com/golang/go/blob/go1.17.1/src/net/http/transport.go#L38-L54

```go
transport  := &http.Transport{
  Proxy: ProxyFromEnvironment,
  DialContext: (&net.Dialer{
    Timeout:   30 * time.Second,
    KeepAlive: 30 * time.Second,
  }).DialContext,
  ForceAttemptHTTP2:     true,
  MaxIdleConns:          100,
  IdleConnTimeout:       90 * time.Second,
  TLSHandshakeTimeout:   10 * time.Second,
  ExpectContinueTimeout: 1 * time.Second,
}
```

しかしこれをコピペして利用するのはリスクがある。Go1.18以降では設定値が変更されるかもしれないし、`http.Transport`型に新たにフィールドが増えた場合未初期化になってしまう。

# `http.DefaultTransport`から少しだけ設定値を変更したオブジェクトを作るには`Clone`メソッドを使う。
`http.Transport`型には`Clone`メソッドというオブジェクトをディープコピーしてくれるメソッドが存在する。
`Clone`メソッドを利用するのが正しい設定のコピーの仕方だろう。この方法ならば、中身が変わったときでもメンテ無しで追従できる。
この手法を利用しているのが、Google APIのGoクライアントだ。
こちらも接続先がGoogle API用のホストに限定されるからか`MaxIdleConnsPerHost`を100に設定している。

https://github.com/googleapis/google-api-go-client/blob/v0.56.0/transport/http/default_transport_go113.go#L12-L21
```go
// clonedTransport returns the given RoundTripper as a cloned *http.Transport.
// It returns nil if the RoundTripper can't be cloned or coerced to
// *http.Transport.
func clonedTransport(rt http.RoundTripper) *http.Transport {
  t, ok := rt.(*http.Transport)
  if !ok {
    return nil
  }
  return t.Clone()
}
```
https://github.com/googleapis/google-api-go-client/blob/v0.56.0/transport/http/dial.go#L163-L171

```go
  // Copy http.DefaultTransport except for MaxIdleConnsPerHost setting,
  // which is increased due to reported performance issues under load in the GCS
  // client. Transport.Clone is only available in Go 1.13 and up.
  trans := clonedTransport(http.DefaultTransport)
  if trans == nil {
    trans = fallbackBaseTransport()
  }
  trans.MaxIdleConnsPerHost = 100
```
# 終わりに
`github.com/googleapis/google-api-go-client` の例は[Goコードリーディングパーティ](https://github.com/basebank/gophers-code-reading-party)で[@dice_zu](https://twitter.com/dice_zu)さんから教えてもらった。  
普段からいろいろなOSSを見ているであろう[@dice_zu](https://twitter.com/dice_zu)さん流石だと思った。見習わなければ。

# 参考
- https://github.com/googleapis/google-api-go-client/blob/v0.56.0/transport/http/default_transport_go113.go#L12-L21
- https://github.com/googleapis/google-api-go-client/blob/v0.56.0/transport/http/dial.go#L163-L171
- https://pkg.go.dev/net/http#Client
- https://pkg.go.dev/net/http#RoundTripper
- https://pkg.go.dev/net/http#Transport.Clone
- https://github.com/basebank/gophers-code-reading-party