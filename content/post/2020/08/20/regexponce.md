+++
title= "[Go] パフォーマンスが出ない正規表現パッケージの使い方をチェックするlinterを作った"
date= 2020-08-20T00:47:56+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = [""]
tags = ["golang","analysis","regexp"]
keywords = ["Go","正規表現","静的解析","analyzer"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

正規表現パッケージのコンパイルを何度も呼び出していないかチェックするlinterを作った。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/regexponce" data-iframely-url="//cdn.iframe.ly/GXXIAnF"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- regexpパッケージのコンパイル処理はプロセス初期化時に一度だけ行うのが望ましい
    - https://golang.org/pkg/regexp
    - `regexp.MustCompile`など
- これをチェックするlinterを作った
    - https://github.com/budougumi0617/regexponce
- `gostaticanalysis/analysisutil`を使えばすぐできた
    - https://github.com/gostaticanalysis/analysisutil
- SSAを利用したが難しい

Goの正規表現を使いたいときは`regexp`パッケージを使う。
このパッケージの使い方には注意すべき点がある。

- https://golang.org/pkg/regexp/

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://developers.eure.jp/tech/golang-regexp/" data-iframely-url="//cdn.iframe.ly/wR6pid0?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

上記記事の以下の部分が注意点だ。

> 正規表現オブジェクトのコンパイルはコストのかかる処理であり、これは以下のようにグローバル領域にコンパイル済みの状態で宣言します。
> 検査時に随時生成することは推奨されません。

たとえば、WebサーバをGoで実装していた場合、リクエストを受け取るたびにコンパイル処理をしているとパフォーマンスが悪い。

```go
func myHandler(w http.ResponseWriter, r *http.Request) {
    // リクエストを受け取るたびに正規表現のコンパイルが走る。
    re := regexp.MustCompile(`(gopher){2}`)
    body, _ := ioutil.ReadAll(resp.Body)
    if re.Match(re) {
        // Do anything...
    }
    // Do anything...
}
```

正しくは次のように書かないといけない。

```go
// グローバルスコープならばmain goroutin起動時に一度だけの実行で済む。
var re = regexp.MustCompile(`(gopher){2}`)

func myHandler(w http.ResponseWriter, r *http.Request) {
    body, _ := ioutil.ReadAll(resp.Body)
    if re.Match(re) {
        // Do anything...
    }
    // Do anything...
}
```

このようなことを静的解析で指摘する静的解析ツールを作った。

と、言っても基のなるロジックは@tenntennさんのcalled linterを踏襲している。
「明らかに一度しか実行されない関数」は正しい利用方法だが、検知できない。
そのため誤検知してしまう部分にたいしてはコメントをつけることで無視する機能もつけた（流用させてもらった）。

- [特定の関数やメソッドの呼び出しを検知するLINTERを作った](https://tenntenn.dev/ja/posts/called/)
- https://github.com/gostaticanalysis/called

また、メインのロジックはanalysisyutilパッケージを使い、作り始めはskeletonコマンドでガッと作っている。便利。

- https://github.com/gostaticanalysis/skeleton
- https://github.com/gostaticanalysis/analysisutil

独自性というところだと、単純に`MuxtCompile`関数などをエラーとせずに次の例外を設けている。

- グローバルスコープ
- `main`関数
    - ただし、`main`関数の中の`for`ループ内で使っていた場合はエラーにする。
- `init`関数

どれも起動時に一度しか実行されない部分なので、静的解析のエラーにはしない。

# 難しかったところ
標準pkgの関数だったので、最初は`go/importer.Default()`経由で`regexp.MustCompile`などの`*types.Func`オブエジェクトを取得していた。
が、同じ関数のはずなのにコード中に現れる`regexp.MustCompile`とはマッチしないようだ。`pos`などの位置も全然違った。
実装を優先したのでちゃんと原因は調べられていない…

# 終わりに

SSAはなかなか難しい（私がデータ構造をきちんと理解できていない）ので、こちらの本を参考に見様見真似で書いた感じ。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://booth.pm/ja/items/1319394" data-iframely-url="//cdn.iframe.ly/afhjP9B"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>



# 参考
- https://github.com/budougumi0617/regexponce
- https://golang.org/pkg/regexp/
- [regexpとの付き合い方 〜 Go言語標準の正規表現ライブラリのパフォーマンスとアルゴリズム〜](https://medium.com/eureka-engineering/regexp%E3%81%A8%E3%81%AE%E4%BB%98%E3%81%8D%E5%90%88%E3%81%84%E6%96%B9-go%E8%A8%80%E8%AA%9E%E6%A8%99%E6%BA%96%E3%81%AE%E6%AD%A3%E8%A6%8F%E8%A1%A8%E7%8F%BE%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E3%81%AE%E3%83%91%E3%83%95%E3%82%A9%E3%83%BC%E3%83%9E%E3%83%B3%E3%82%B9%E3%81%A8%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0-984b6cbeeb2b)
- [特定の関数やメソッドの呼び出しを検知するLINTERを作った](https://tenntenn.dev/ja/posts/called/)
- https://github.com/gostaticanalysis/called
- https://github.com/gostaticanalysis/analysisutil
- [逆引きGoによる静的解析入門](https://booth.pm/ja/items/1319394)