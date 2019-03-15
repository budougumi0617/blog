+++
title= "[Go] selectのcase文中でch <- <- chやch <- f()をしない方が良い #golangjp #横浜go読書会"
date= 2019-03-15T19:41:58+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "select", "channel", "goroutine"]
keywords = ["Go", "並行処理","select", "channel"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


簡潔にまとめられなかったので式表現をそのままタイトルに書いてしまったが、分かっていないとエンバグしそうな挙動を見つけたのでメモしておく。

<!--more-->

# TL;DR
- チャネルを使ったselectのcase文中で、関数を呼び出した結果でチャネルへの送信をしないほうがよい
- チャネルを使ったselectのcase文中で、別のチャネルから受信した結果でチャネルへの送信をしないほうがよい

文章で書くとわかりにくいのだが、コードで書くと以下のような処理を書くのは避けたほうがよい、というのが主題だ。

```go
 select {
 case ch1 <- f(): // チャネルへ関数を呼び出した結果を送信する
 case ch1 <- <- ch2: // チャネルへ別のチャネルからの受信結果を送信する
 }
```
なお、検証環境は以下。

- `go version go1.12 darwin/amd64`
- https://play.golang.org/ （2019/03/15時点）

# チャネルを使ったselectのcase文中で、関数を呼び出した結果でチャネルへの送信をしないほうがよい
まずselect文のcaseで関数呼び出しをしているときを検証した。  
以下の処理は`done`チャネルに通知が入るまで無限ループが行われるコードだ。  
`ch1`や`ch2`の準備ができているときは`f`関数の戻り値を送信する。このプログラムを実行したとき、`called f`という文字列を何回出力するだろうか？（`f`関数は何回実行されるだろうか？）

- https://play.golang.org/p/ZvypfcE-oPe

```go
package main

import (
	"fmt"
)

func f() int {
	fmt.Println("called f")
	return 1
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	done := make(chan struct{})

	go func() {
		fmt.Printf("f return = %d\n", <-ch1)
		done <- struct{}{}
	}()

	for {
		select {
		case <-done:
			fmt.Println("done")
			return
		case ch1 <- f():
		case ch2 <- f():
		}
	}
}
```

私は`fmt.Printf("f return = %d\n", <-ch1)`の時に`ch1`から値を受け取るときだけ`f`関数が呼び出されると思ったので1回と予想していた。  
だが、答えは4回だ。

```
called f
called f
f return = 1
called f
called f
done
```

どうやら**`select`文に突入した時点でチャネルへの送信文(`channel <- XXXX`)の右辺(`XXXX`)が実行されなかった`case`も含めて確定的に行われてしまう**らしい。  
この挙動はselectのcase文でチャネルからの受信式(`<-channel`)を使っても同様に発生する。


# チャネルを使ったselectのcase文中で、別のチャネルから受信した結果でチャネルへの送信をしないほうがよい
以下のコードは別のチャネル（`fchX`）から送信された`int`を受信するチャネル（`chX`）を複数`select`文で並べて待機するコードだ。

- https://play.golang.org/p/cdZORmhc8k7

```go
package main

import (
	"fmt"
)

func main() {
	sample3()
}

func sample3() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	done := make(chan struct{})

  // 1ずつインクリメントされたintを送信するチャネルを生成する関数
	f := func() <-chan int {
		ch := make(chan int)
		go func() {
			var i int
			for {
				ch <- i
				i++
			}
		}()
		return ch
	}

	go func() {
		fmt.Printf("fch1 return = %d\n", <-ch1)
		fmt.Printf("fch2 return = %d\n", <-ch2)
		fmt.Printf("fch1 return = %d\n", <-ch1)
		done <- struct{}{}
	}()

	fch1 := f()
	fch2 := f()
	for {
		select {
		case ch1 <- <-fch1:
		case ch2 <- <-fch2:
		case <-done:
			fmt.Println("done")
			return
		}
	}
}
```

この実行結果は以下となる。

```
fch1 return = 0
fch2 return = 1
fch1 return = 2
done
```

標準出力している部分のコードを確認する。

```go
fmt.Printf("fch1 return = %d\n", <-ch1)
fmt.Printf("fch2 return = %d\n", <-ch2)
fmt.Printf("fch1 return = %d\n", <-ch1)
```

最初の`fmt.Printf("fch1 return = %d\n", <-ch1)`の結果は`0`だ。`ch1`への入力の`fch1`（`f()`で生成したチャネル）は`0`からインクメントされた整数を返すチャネルなので、期待通りだ。
だが、`fmt.Printf("fch2 return = %d\n", <-ch2)`は`ch2`(`fch2`)から初めて値を取り出すので`0`を期待したいところだが、`1`が出力される。
また、二回目の`fmt.Printf("fch1 return = %d\n", <-ch1)`では`ch1`(`fch1`)に対する二度目の取得なので、`1`が出てくるのを期待するのだが、`2`が出てくる。

`select`で実行されたなかった時でも`case`中の右辺が評価されてしまうので、
**`fch2`から`ch2`へ渡されたはずの`0`や、`fch1`から`ch1`に渡されたはずの`1`はどこか虚空に消えてしまった**ようだ…

# 最後に
`select`の`case`中に関数呼び出しやチャネルからの受信をしていると思わぬ挙動をしてしまうことをまとめた。  
今回の内容は[第25回横浜Go読書会](https://yokohama-go-reading.connpass.com/event/120344/)に参加中に話題になったことをまとめた内容である。
具体的な仕様や実装までは追えなかったので実動作についてのみのまとめになってしまった。  
本当ならば実際にGoの実装のどの部分でこのような挙動を実現しているかなどちゃんと調べていきたい。

- 横浜Go読書会
  - https://yokohama-go-reading.connpass.com/
- Select statements | The Go Programming Language Specification
  - https://golang.org/ref/spec#Select_statements


