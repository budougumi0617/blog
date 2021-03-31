+++
title= "[Go] stringの比較でヌルポのpanicが発生する（こともある） #横浜Go読書会"
date= 2021-03-31T11:00:23+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","gotips"]
keywords = ["Go"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

誤った並行処理の実装をしていると、`string`の比較でもヌルポのセグフォが発生する。  
正しく実装していればお目にかかることはないが、とても学びになったのでメモしておく。

<!--more-->

# TL;DR
- Goの`string`はプリミティブな型でポインタ参照はしていない（ように操作できる）
- が、誤った並行処理を行っていると、ヌルポのpanicが発生する
    - `panic: runtime error: invalid memory address or nil pointer dereference`
- runtime上で`string`は`stringStruct`型で長さと具体的な値へのポインタを持つ
    - https://github.com/golang/go/blob/go1.16.2/src/runtime/string.go#L228-L231
- 並行処理の実装が誤っていると、長さを有した状態でヌルポインタなオンメモリデータが生成される
    - CPUのレジスタあるいはキャッシュの状態がメモリに半端な状態で反映されるため

サンプルコードは以下。

https://play.golang.org/p/lD5-kBCwET4

```go
package main

func main() {
        hello()
}

var a string

func hello() {
        go func() { a = "hello" }()
        for a != "hello" {
        }
        print(a)
}
```

play ground上でも何回か実行すると実際に文字列`a`変数を`a != "hello"`と比較しているところで以下のような結果が得られる。

```
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x45eb46]

goroutine 1 [running]:
main.hello()
	/tmp/sandbox884085650/prog.go:11 +0x46
main.main()
	/tmp/sandbox884085650/prog.go:4 +0x25
```

# stringなのにnil pointer dereference
横浜Go読書会でGoのメモリモデルの勉強をしていた。

- The Go Memory Model
    - https://golang.org/ref/mem

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://yokohama-go-reading.connpass.com/event/206437/" data-iframely-url="//cdn.iframe.ly/XYG9EVf"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>



発端は忘れたが、誤った並行処理のサンプルコードを書いていたら文字列の比較でヌルポが発生してしまった。  
サンプルコードはTL;DRに書いた先の通り。

# メモリモデルと並行処理

資料を読む中で柴田（[@yoshiki_shibata](https://twitter.com/yoshiki_shibata))さんから以下の補足があった。

> f()を実行するゴルーチンとg()を実行するゴルーチンが、別々のOSスレッド上で実行された場合、コンパイラによる実行命令の入れ替え、あるいはout-of-order CPUのreorder bufferの影響でg()関数内で a をメ モリから読み出したときに、f()関数内でのa = 1の書き込みはメモリに反映されていない可能性があります。

また、メモリモデル本文にも次のような一節がある。

> If the channel were buffered (e.g., c = make(chan int, 1)) then the program would not be guaranteed to print "hello, world". (It might print the empty string, crash, or do something else.)

（チャネル操作を間違えると文字列の値が不定になったりクラッシュする）

ざっくりいうと、CPUのコアはコアごとにキャッシュやレジスタを持っている。キャッシュやレジスタは個別のコアのみアクセス可能である。  
なので、マルチコアCPUの場合、その下の層のメモリにまでコアの処理結果が反映されてないと他のコアの変更結果が見えない。

- https://nipponkaigi.net/wiki/Re-order_buffer
- https://nipponkaigi.net/wiki/Out-of-order_execution

ここで、CPUは必ずしも処理の順でメモリに結果を反映するわけではない。   
また、他のゴルーチンがアクセスする際に、メモリへの反映が正しく完了している保証もない[^rdb]。  
では、それが`string`の参照でヌルポが発生するのとどんな関係があるのだろうか？

[^rdb]: RDBでトランザクションを貼る状況をイメージすればよい

# runtime上での実体確認
まず、runtime上の`string`型は`stringStruct`という長さと参照ポインタをもった構造体として処理される。

- https://github.com/golang/go/blob/go1.16.2/src/runtime/string.go#L228-L231

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```

この構造体の動きは`reflect#StringHeader`で確認することができる。

- https://golang.org/pkg/reflect/#StringHeader

先述のコードに`reflect#StringHeader`を使って`stringStruct`の状態を出力しつづけるようにしたのが次のコードだ。

https://play.golang.org/p/1oCesyqPjh-

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	hello()
}

var a string

func hello() {
	sh := (*reflect.StringHeader)(unsafe.Pointer(&a))
	go func() {
		for ;a != "hello?" ;{
			fmt.Printf("%#v\n", sh)
		}
	}()
	go func() { a = "hello" }()
	for a != "hello" {
	}
	print(a)
}
```

このコードは何も排他制御をしていないため、`hello`関数内のゴルーチンが`a = "hello"`と`stringStruct`を操作している。
そして、その結果が正しくメモリへ反映されるかされていないかにかかわらず `a != "hello"` といった参照を行う。その結果`stringStruct.len`の長さが`!0`なのに`stringStruct.str`が`nil`の状態が発生していると考えられる。

出力としては `&reflect.StringHeader{Data:0x4bcccf, Len:5}` というデータも取れているが、
実際はメモリ上で`Len != 0`の状態でも`Data`フィールドは`nil`の状態になってしまったのだと思われる。

```
&reflect.StringHeader{Data:0x0, Len:0}
&reflect.StringHeader{Data:0x0, Len:0}
&reflect.StringHeader{Data:0x0, Len:0}
&reflect.StringHeader{Data:0x4bcccf, Len:5}
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x498ece]

goroutine 1 [running]:
main.hello()
	/tmp/sandbox091726292/prog.go:23 +0x6e
main.main()
	/tmp/sandbox091726292/prog.go:10 +0x25
```

# 終わりに
以前はHPCや組み込み系のプログラムを書いていたので、久しぶりにCPUなどを意識した気がする。  
普段Webサービスを作る上でCPU上のデータの挙動やメモリモデルまで気にしないが、いつかなにかのデバッグとかで役に立つかなと思う[^debug]。

[^debug]: こんなところまで見ないといけないバグには遭遇したくないが

また、今回のようなサンプルコードを動かしてみてやっと一見当たり前のことしか書いていない事前発生（*happens before*）の意味がわかってきた気がする。

>To specify the requirements of reads and writes, we define happens before, a partial order on the execution of memory operations in a Go program. If event e1 happens before event e2, then we say that e2 happens after e1. Also, if e1 does not happen before e2 and does not happen after e2, then we say that e1 and e2 happen concurrently.


詳細は忘れたとしても次の理解だけでもしておくほうがよさそうだ。

- 並行処理を書くときは正しく排他制御をしておくこと
- 文字列型の処理行でセグフォが出たら並行処理の排他制御を疑うこと



余談だが、メモリモデルの文書の冒頭にも「この文書の理解が必要になる"賢い"プログラミングするな」と注意書きがある（とはいえ書かなくても理解しているのが望ましいだろう）。

> If you must read the rest of this document to understand the behavior of your program, you are being too clever.
> Don't be clever.

# 参考
- https://yokohama-go-reading.connpass.com/event/206437/
- https://golang.org/ref/mem
- https://nipponkaigi.net/wiki/Re-order_buffer
- https://nipponkaigi.net/wiki/Out-of-order_execution