+++
title= "Go1.16で追加されたio#ReadAll関数から読むストリーミング中のバッファ拡張の仕方"
date= 2021-02-22T08:00:00+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","slice","go116","gotips"]
keywords = ["Go","スライス","append"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

実質Futureさんの記事の引用なんだけれど自分用メモ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://future-architect.github.io/articles/20210210/index.html" data-iframely-url="//cdn.iframe.ly/gbhA0jg"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- Go1.16で`io/ioutil#ReadAll`関数が`io#ReadAll`関数に書き直された
- `io#ReadAll`関数は`bytes#Buffer`型を利用せずにバッファ管理を行なっている
    - `b = append(b, 0)[:len(b)]`
- `append`を使ったDRYなコード。`append`の性能向上や最適化の恩恵に預かれるすごいコード
    - ただ自分で使う機会はあまりないかもしれない

# Go1.16で追加された`io#ReadAll`
先日リリースされたGo1.16では`io/ioutil#ReadAll`関数が`io#ReadAll`関数に書き直された。

- Go 1.16 is released - The Go Blog
    - https://blog.golang.org/go1.16

`ReadAll`関数は`io.Reader`インターフェイスからすべてのデータを読み出す関数だ。

```go
func ReadAll(r Reader) ([]byte, error)
```

移植された`ReadAll`関数は内部の実装がすべて書き換えられている。  
Go1.15以前の`io/ioutil#ReadAll`関数は`bytes`パッケージに依存していたが、`bytes`パッケージは`io`パッケージを`import`しており循環参照になるため利用できないからだ。

https://github.com/golang/go/blob/go1.16/src/bytes/buffer.go#L9-L13

```go
// bytes/buffer.go の冒頭
import (
	"errors"
	"io"
	"unicode/utf8"
)
```

# `io/ioutil#ReadAll`の実装
[@rsc](https://github.com/rsc)さんによって追加されたCLは次のCLになる。

- https://go-review.googlesource.com/c/go/+/263141

該当コードはこちら。

https://github.com/golang/go/blob/go1.16/src/io/io.go#L622-L642

```go
// ReadAll reads from r until an error or EOF and returns the data it read.
// A successful call returns err == nil, not err == EOF. Because ReadAll is
// defined to read from src until EOF, it does not treat an EOF from Read
// as an error to be reported.
func ReadAll(r Reader) ([]byte, error) {
	b := make([]byte, 0, 512)
	for {
		if len(b) == cap(b) {
			// Add more capacity (let append pick how much).
			b = append(b, 0)[:len(b)]
		}
		n, err := r.Read(b[len(b):cap(b)])
		b = b[:len(b)+n]
		if err != nil {
			if err == EOF {
				err = nil
			}
			return b, err
		}
	}
}
```

この中の`b = append(b, 0)[:len(b)]`が凄まじいなと思った。

# `append`関数を利用してバッファを拡張する
最初読んだとき何しているのかわからなかったが、ここでやっているのは次の処理だ。

1. 1バイトだけ追加する`append`関数を実行する
    -  直前の`len(b) == cap(b)`より、バッファはキャパシティ不足
1. バッファのキャパシティが`append`関数によって拡張され`0`が追加される
1. `0`がひとつだけ増えたバッファだが`[:len(b)]`によって元の長さのスライスになる。


以下の点で「さすがすぎる…」という感想を持った。

- たった一行のスマートなコード
- ポインタいじったりカーネルに近い操作をしているわけではないシンプルなコード
- パット見よくわからないがちゃんと解説コメントが書いてある
- DRY
- `append`関数に対するパフォーマンスチューニングや最適化に依存できる

一度みたら書けるが自分ではこのやり方は思いつかないなと思った。

# 終わりに
同様な状況を自分で実装するとき、大抵は`bytes#Buffer`型を利用しているか確保すべきバッファサイズが自明だろう。
よってなかなか独自型でストリームデータを扱うことはないかもしれないが、なにかあったときはスッと実装したいイディオムだった。  

というかGo1.16全然触れていないので早く触らなければ！

## 余談

標準パッケージもpkg.go.devで見るようになった認識だ。
しかしpkg.go.devだと追加されたバージョンがわからないので、まだgolang.org/pkg見ていたほうがいいのかな？

- https://pkg.go.dev/io#ReadAll
- https://golang.org/pkg/io/#ReadAll

# 参考
- https://golang.org/pkg/io/#ReadAll
- [Go1.16からのio/ioutilパッケージ](https://future-architect.github.io/articles/20210210/)
