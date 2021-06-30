+++
title= "Goでプログラムを終了するときのお作法"
date= 2021-06-30T17:57:32+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","server","cli"]
keywords = ["終了方法","exit","Go","Go言語"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goにはいくつかプログラムを終了させる手段が存在する。  
プログラムを終了させるときにどれを選べばいいか調べてみた。結論から言うと`main`関数内で`defer`を使わず`os.Exit`関数を呼ぶ。

<!--more-->

# TL;DR
- Goで意図的にプログラムを終了させることができる処理は次のとおり
    - https://golang.org/pkg/os/#Exit
    - https://golang.org/pkg/builtin/#panic
    - https://golang.org/pkg/runtime/#Goexit
- `main`関数内で`defer`を使わず`os.Exit`関数を呼ぶのが一番良いと思う
- `panic`関数は終了コードを明示的に決めることができない
    - 一方ですべてのゴルーチンを停止してくれるらしい
- `runtime#Goexit`関数を`main`関数で呼ぶのは行儀が悪そう
    - > Calling Goexit from the main goroutine terminates that goroutine without func main returning. 
    - `defer`宣言済みの関数に関してはすべて実行してから終了してくれるらしい
- `os#Exit`関数は任意の終了コードでプログラムをただちに終了することができる
    - `defer`宣言済みの関数は実行されない

なお、すでにまとめている方のブログ記事も存在する。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://sharpknock.com/posts/programming/golang-exit.html" data-iframely-url="//cdn.iframe.ly/NyGpegI"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

また、この記事の調査内容は2021年6月時点で最新のGo1.16時点の各仕様である。

# Goのプログラムを終了させる
私はGoのプログラムを終了させるときはいつも`os#Exit`関数を使っていた。

- https://golang.org/pkg/os/#Exit

「`defer`ちゃんと動くし`panic`じゃだめなんですか？」という質問を受けたときちゃんと答えられなかったので仕様を調査してみた。

# Goでプログラムを終了させる方法
`main`関数をそのまま終了する、セグフォなどで異常終了する、などを除いて、コードの表現としてプログラムを終了させる方法は主に3つある。

- `os#Exit`関数で終了する
- `panic`関数で終了する
- `runtime#GoExit`関数で終了する

それぞれの仕様を確認してみた。

## `os#Exit`関数で終了する
- https://golang.org/pkg/os/#Exit

一番オーソドックスな方法だと思っているが、仕様をみると、`defer`済みの関数については実行されないと記載があった。

> The program terminates immediately; deferred functions are not run.

なので、`main`関数で`defer`文と併用して`os#Exit`関数を実行すると意図しない挙動をする可能性がある。また、「immediately」なので他のゴルーチンの終了などは待機してくれない。

## `panic`関数で終了する
- https://golang.org/pkg/builtin/#panic

調査前は`panic`関数で終了するほうがめちゃくちゃな終了をすると思っていた。
しかし仕様を呼んでみると意外と`defer`済みの関数などはちゃんと呼ばれるようだ。

> When a function F calls panic, normal execution of F stops immediately. Any functions whose execution was deferred by F are run in the usual way, and then F returns to its caller. 

しかし、終了コードをプログラマが指定することはできない。

## `runtime#GoExit`関数で終了する
- https://golang.org/pkg/runtime/#Goexit

直接使ったことはないが、`runtime#GoExit`関数でもプログラムを終了することができる。`testing`パッケージの`Fatal`関数を呼ぶと内部で実行されるようだ。

https://github.com/golang/go/blob/go1.16.5/src/testing/testing.go#L710-L742

仕様を読むと、`main`関数で呼ぶ場合、怪しい挙動をするようなのでこれはプログラムを終了させるときに使わないほうがよさそうだ。

> Calling Goexit from the main goroutine terminates that goroutine without func main returning. Since func main has not returned, the program continues execution of other goroutines. If all other goroutines exit, the program crashes.

## 結論
`defer`の利用に注意しながら`main`関数で`os#Exit`関数を実行するのがよさそうだった。  
コマンドラインならばたとえばこんな感じでよいのではないだろうか。

```go
func main() {
  os.Exit(run(os.Args, os.Stdin, os.Stdout, os.Stderr))
}

func run(args []string, inStream io.Reader, outStream, errStream io.Writer) int {
    // do anything...
}
```

# 終わりに（余談）
`panic`のような予約語の機能の説明はどこを見ればよいのかわかっていなかった。  
`builtin`パッケージを見ると`panic`を始め、`cap`や`append`と言ったビルドイン関数の明文化された仕様を読むことができるのを知れてよかった。

- https://golang.org/pkg/builtin

# 参考
- https://golang.org/pkg/os/#Exit
- https://golang.org/pkg/builtin/#panic
- https://golang.org/pkg/runtime/#Goexit
- https://sharpknock.com/posts/programming/golang-exit.html