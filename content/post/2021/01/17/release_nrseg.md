+++
title= "GoのアプリにNew Relic APMを導入する時とても便利なCLIを作った"
date= 2021-01-17T10:00:33+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","o11y","newrelic"]
keywords = ["newrelic","o11y","go"]
twitterImage = "logos/newrelic.png"

+++

GoのWebアプリコードにNew Relic APMでSpan（Segment）を計測するコードを自動挿入するOSSを作った。  
New RelicをGoのアプリケーションへ導入するときに利用をオススメする。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/nrseg" data-iframely-url="//cdn.iframe.ly/HrUY585?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- New Relic APMは分散トレーシングなどにも対応しているAPM
- Spanを計測するためには、関数ごとにSegmentの処理を差し込む必要がある
- 指定ディレクトリ以下にあるGoのコードにFunction segmentsを自動挿入するOSSを作った
- https://github.com/budougumi0617/nrseg


# New Relic APM
New Relic APMはNew Relic Oneに含まれている分散トレーシングなどにも対応したモニタリングツールだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://newrelic.com/jp/products/application-monitoring" data-iframely-url="//cdn.iframe.ly/tp2FaR3?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

個人でも業務でも使い始めた[^base_nr]のだが、エージェントを挿入すれば簡単にアプリケーションを丸裸にできる。

[^base_nr]: https://newrelic.com/jp/press-release/20201217

## GoのWebアプリケーションにNew Relic APMを導入する手順
APMだけならば導入もすぐ完了する。

- Install New Relic for Go
    - https://docs.newrelic.com/docs/agents/go-agent/installation/install-new-relic-go

ざっくりとまとめると、まずアプリ起動時にNew Relicと接続するための`Application`オブジェクトを生成する。

```go
app, err := newrelic.NewApplication(
    newrelic.ConfigAppName("Your Application Name"),
    newrelic.ConfigLicense("__YOUR_NEW_RELIC_LICENSE_KEY__"),
)
```

そして各種ハンドラーのエンドポイントを次のようにラップする。
```go
http.HandleFunc(newrelic.WrapHandleFunc(app, "/users", usersHandler))
```

主要なフレームワーク、MySQLなどならば専用のラッパーも用意されている。

- https://github.com/newrelic/go-agent/tree/master/v3/integrations

## Spanの計測が大変

これでNew Relic上からTraceを確認できるようになる。  
ここまでの対応ならば、すぐにできるだろう。しかし、ここまでだとTraceは取得できるようになっても、Span[^span]が取得できない。

[^span]: 1つのTraceの中にあるオペレーションの1つ１つの実行処理時間などを計測したもの

- Instrument Go transactions1
    - https://docs.newrelic.com/docs/agents/go-agent/instrumentation/instrument-go-transactions
- Instrument Go segments
    - https://docs.newrelic.com/docs/agents/go-agent/instrumentation/instrument-go-segments

Goのアプリケーション上でSpanを取得するには、関数やメソッドごとにFunction segmentsを明示的に埋め込んでいく必要がある。

```go
defer txn.StartSegment("mySegmentName").End()
```

これを中規模以上の既存のアプリケーションに実施しようとすると、**関数/メソッド1つ1つを修正していく必要**があり非常に面倒なことになる。

# nrseg
上記の面倒くささを解消するためのコマンドラインツールがnrsegだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/nrseg" data-iframely-url="//cdn.iframe.ly/HrUY585?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Macの場合はBrewでインストールすることができる。

```bash
$ brew install budougumi0617/tap/nrseg
```

このような既存のコードがあったとする。

```go
package input

import (
  "context"
  "fmt"
  "net/http"
)

type FooBar struct{}

func (f *FooBar) SampleMethod(ctx context.Context) {
  fmt.Println("Hello, playground")
  fmt.Println("end function")
}

func SampleFunc(ctx context.Context) {
  fmt.Println("Hello, playground")
  fmt.Println("end function")
}

func SampleHandler(w http.ResponseWriter, req *http.Request) {
  fmt.Fprintf(w, "Hello, %q", req.URL.Path)
}

// nrseg:ignore you can be ignored if you want to not insert segment.
func IgnoreHandler(w http.ResponseWriter, req *http.Request) {
  fmt.Fprintf(w, "Hello, %q", req.URL.Path)
}
```

上記のコードを含んだディレクトリにnrsegを実行する。

```bash
$ nrseg .
```

そうすると、上記のようなコードが次のようになる。

```go
package input

import (
  "context"
  "fmt"
  "net/http"

  "github.com/newrelic/go-agent/v3/newrelic"
)

type FooBar struct{}

func (f *FooBar) SampleMethod(ctx context.Context) {
  defer newrelic.FromContext(ctx).StartSegment("foo_bar_sample_method").End()
  fmt.Println("Hello, playground")
  fmt.Println("end function")
}

func SampleFunc(ctx context.Context) {
  defer newrelic.FromContext(ctx).StartSegment("sample_func").End()
  fmt.Println("Hello, playground")
  fmt.Println("end function")
}

func SampleHandler(w http.ResponseWriter, req *http.Request) {
  defer newrelic.FromContext(req.Context()).StartSegment("sample_handler").End()
  fmt.Fprintf(w, "Hello, %q", req.URL.Path)
}

// nrseg:ignore you can be ignored if you want to not insert segment.
func IgnoreHandler(w http.ResponseWriter, req *http.Request) {
  fmt.Fprintf(w, "Hello, %q", req.URL.Path)
}
```

`nrseg:ignore`とコメントした関数以外の**すべての関数/メソッドに対してSpanを生成するためのNew Relicライブラリの呼び出しが自動挿入**される。  
nrsegコマンドはディレクトリの下のすべてのGoコード[^testdata]に対して上記の修正を行なう。
既存アプリケーションのコード量が多ければ多いほど便利！

[^testdata]: testdataディレクトリや`-i`オプションで指定されたディレクトリは除く

# 技術的なところ
やっていることはディレクトリをトラバースしながら各コードのASTに無理やり該当コードを挿入してファイルを更新している。  
挿入時にシグネチャの中にある`context.Context`の変数名を取得したり、細かいASTのパースも行なっている。  
めんどくさかったところはおいおい個別の記事でメモを公開したいと思う。

# 終わりに
今年の目標（[2020年のTry](/2020/12/27/retrospective-2020/#try)）として「`ひとつ突き詰めて実装してみる`」を掲げている。
`nrseg`はGoのASTと仲良くなれるし業務で利用しはじめたため、目標とマッチしつつモチベーションも保ちやすい。  
また、New Relicを使って今年のISUCONを出ることを想定して「（サブミット時に）挿入したコードをすべて削除する」サブコマンドなども用意したいと思う。
引き続き機能強化していきたい。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/nrseg" data-iframely-url="//cdn.iframe.ly/HrUY585?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 参考資料
- https://github.com/budougumi0617/nrseg
- Install New Relic for Go
    - https://docs.newrelic.com/docs/agents/go-agent/installation/install-new-relic-go
- Instrument Go segments
    - https://docs.newrelic.com/docs/agents/go-agent/instrumentation/instrument-go-segments
- [2020年振り返り](/2020/12/27/retrospective-2020/#try)
