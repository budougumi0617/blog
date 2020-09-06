+++
title= "[Go] signal.Notifyを使うときは必ずバッファ付きチャネルで利用すること"
date= 2020-09-06T14:42:42+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","gotips","signal"]
keywords = ["Go","select","signal.Notify","goroutine"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


Goで`singal.Notify`関数を使うときは必ずバッファありチャネルを利用しなくてはいけない。  
なぜバッファなしチャネルを使ってはいけないのかまとめた。

- https://godoc.org/os/signal#Notify

<!--more-->

# TL;DR
- `singal.Notify`関数を使うときは必ずバッファありチャネルを利用しなくてはいけない
- 仕様にも明記されている
    - https://godoc.org/os/signal#Notify
- シグナル処理はWebサービス開発時にも必要
    - https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/update-service.html
    - https://kubernetes.io/ja/docs/concepts/workloads/pods/pod/
- バッファなしチャネルを使うと、シグナルを取りこぼす可能性がある

# singal.Notify関数
Goでシグナルハンドリングをするときに利用するのが`singal.Notify`関数だ。

```go
func Notify(c chan<- os.Signal, sig ...os.Signal)
```

この関数は第1引数にシグナルを受信するためのチャネルを指定し、第2引数以降はハンドリングしたいシグナルを羅列していく。  
このときチャネルはバッファありチャネルでないといけない。  
これは`singal.Notify`関数の仕様に明記されている。

- https://godoc.org/os/signal#Notify

> Package signal will not block sending to c: the caller must ensure that c has sufficient buffer space to keep up with the expected signal rate. For a channel used for notification of just one signal value, a buffer of size 1 is sufficient.

# シグナルハンドリングについて
シグナルは身近なところでいうとコマンドラインツールをコントロールCLIなどで終了したり、フリーズしたソフトやプロセスに対してキルコマンドを実行したときに発生する。  
ただ、Webサービスを作る上でもシグナルを意識する必要はある。  
たとえばECSやk8s上で起動するコンテナはコンテナ外部からシグナルを受信したとき適切に終了する必要がある。  
シグナルをハンドリングしてグレースフルシャットダウンをする実装をしていないと、処理の終了を待たずにコンテナが強制遮断されうる。

- https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/update-service.html
- https://kubernetes.io/ja/docs/concepts/workloads/pods/pod/

# チャネルの仕様の簡単な確認
Goのチャネルにはバッファありチャネルとバッファなしチャネルが存在する。  
~~バッファなしチャネルを介した場合、何もしないと送信側goroutine、受信側goroutineの間には*happens before*の関係が成り立つ。~~

- Happens Before | The Go Memory Model
    - https://golang.org/ref/mem#tmp_2

~~これは受信側goroutineが受信を完了するまで、送信側goroutineが再度起動されないことを意味する。~~


## 2020/09/06 20:12 追記
*happens before*はあくまで変数の可視性の話なのでゴルーチンの動きを説明するものではない。と指摘していただきました。  
チャネルの挙動を決めているのは言語仕様でした。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">happens beforeですが、バッファなしチャネルの動きを説明するものではなく、変数への更新が別のゴールチーンから見えるか否かに影響を与えるものなので、ブログの説明が少し違っていると思います。</p>&mdash; Yoshiki Shibata/柴田芳樹 (@yoshiki_shibata) <a href="https://twitter.com/yoshiki_shibata/status/1302527258599370752?ref_src=twsrc%5Etfw">September 6, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

バッファなしチャネルを介した場合、送信は受信側が受信を開始するまで成功しない（処理が完了しない）。

> If the capacity is zero or absent, the channel is unbuffered and communication succeeds only when both a sender and receiver are ready.

- Channel types | The Go Programming Language Specification
	- https://golang.org/ref/spec#Channel_types

> A send on an unbuffered channel can proceed if a receiver is ready. 

- Send statements | The Go Programming Language Specification
	- https://golang.org/ref/spec#Send_statements

（追記終わり。）

言い換えると、受信が完了するまで送信側では送信が行われず処理を再開できない。

```go
ch := make(chan int)

ch <- 10 // どこかで受信されるまで処理が進まない
```

# signal.Notifyでバッファありチャネルを使わないといけない理由
ある行儀の悪いバッファなしチャネルがシグナルを待機（受信）していたとする。
```go
c := make(chan os.Signal) // バッファなしの行儀が悪いチャネル
signal.Notify(c, os.Interrupt, os.Kill)
```

シグナルが発生したとき、このバッファなしチャネルが受信するまで送信側goroutineの処理が止まってしまってはいけない。  
他のチャネルも受信待機しているかもしれないので適切なシグナルハンドリングができなくなってしまう。  
そのため、`signal`パッケージのシグナル送信側の処理は冒頭で引用した仕様の通り、ノンブロッキングな送信を行う。

> Package signal will not block sending to c

具体的なコードを見ると、まずsignalパッケージは以下のようなループでシグナルを処理している。
（[signal_recv](https://github.com/golang/go/blob/a538b59fd2428ba4d13f296d7483febf2fc05f97/src/runtime/sigqueue.go#L122-L125)）は私にはちょっとむずかしいので説明省略）

https://github.com/golang/go/blob/a538b59fd2428ba4d13f296d7483febf2fc05f97/src/os/signal/signal_unix.go#L21-L25
```go
func loop() {
	for {
		process(syscall.Signal(signal_recv()))
	}
}
```

`process`関数の中をみてみると、`singal.Notify`関数で登録されたチャネルそれぞれにシグナルを送信している。  
このとき、チャネルが送信ブロック状態だった場合は`default`節があるので**そのチャネルを無視して次のチャネルへシグナルを送信する**。

https://github.com/golang/go/blob/a538b59fd2428ba4d13f296d7483febf2fc05f97/src/os/signal/signal.go#L240-L248
```go
	for c, h := range handlers.m { // cがsignal.Notifyの第一引数のチャネル
		if h.want(n) {
			// send but do not block for it
			select {
			case c <- sig:
			default:
			}
		}
	}
```

このため、バッファなしチャネルを使って`signal.Notify`関数を使った場合、チャネルが受信待機するまえに送信されたシグナルは検知できなくなる。


# おわりに
`signal.Notify`関数はバッファなしチャネルで利用してはいけない。  
これは`signal.Notify`関数の仕様にも明記されている。  
今回はドキュメントの注意書きを無視した場合どのような懸念事項が発生するのか確認することができた。

# 参考
- https://godoc.org/os/signal#Notify
- [Happens Before | The Go Memory Model](https://golang.org/ref/mem#tmp_2)
- [Channel types | The Go Programming Language Specification](https://golang.org/ref/spec#Channel_types)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=b23c76a9208ccfad7862a4ffc8269211"></iframe>