+++
title = "Loop by Time Tick in Golang"
date = 2018-11-18T11:53:10+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "time"]
+++

[Go言語による並行処理](http://amazon.jp/dp/4873118468)に知らない書き方があったのでメモ。

<!--more-->

# TL;DR
- `time.Tick()`を`for range`で使うとスマートに無限ループが書ける
- x秒間だけn秒おきになにかするループは`time.Tick()`使わないほうがシンプルな気がする


# n秒おきになにかする無限ループ

## forループとtim.Sleep()で書くとき

x秒おきに何かをする無限ループを書くときこんな`for`ループを書きがちだった。
（playgroundのサンプルコードは途中で終わるようにカウンタを入れている。）

https://play.golang.org/p/Do7e6enUu5z

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	for {
		fmt.Println("Tick!!")
		time.Sleep(3 * time.Millisecond)
	}
}
```


## time.Tickを使ってX秒おきに無限ループ
`time.Tick`を使えば以下のようにも書ける。

https://play.golang.org/p/VlWP05XLRGX
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	for range time.Tick(3 * time.Millisecond) {
		fmt.Println("Tick!!")
	}
}
```

# x秒間だけn秒おきになにかする有限ループ

特定の時間の間だけ定期的に何かをするサンプルコードもGo言語による並行処理に載っていた。

https://play.golang.org/p/O4KWXe5KQJJ
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	limit := 5 * time.Millisecond
	// time.Since()で取得したDurationでループを止めるか決める
	for begin := time.Now(); time.Since(begin) < limit; {
		fmt.Println("Tick!!")
		time.Sleep(1 * time.Millisecond)
	}
}
```

`time.Tick`を使うとこんな感じ？これは素直に上の書き方をしたほうがシンプルだとおもう。

https://play.golang.org/p/8NxMqbqcmX5
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	limit := 5 * time.Millisecond
	begin := time.Now()
	for now := range time.Tick(1 * time.Millisecond) {
		fmt.Println("Tick!!")
		// time.Tickで取得した現在時間とループ開始直前の時間の差分でループを止めるか決める
		if now.Sub(begin) >= limit {
			break
		}
	}
}

```

# 終わりに

`time.Tick()`は内部的には`time.Ticker`を生成している。
`time.Ticker`を使うときは明示的に`Stop()`を呼ばないとリークするらしいが、`time.Tick()`を`range`で使っている場合はループのスコープが外れればGCされるのかな？

https://golang.org/src/time/tick.go?s=1752:1785#L44

```go
func Tick(d Duration) <-chan Time {
	if d <= 0 {
		return nil
	}
	return NewTicker(d).C
}
```

# 参考
- Goで一定周期で何かを行う方法
  - https://qiita.com/ruiu/items/1ea0c72088ad8f2b841e
- time.Tick() を for ループの中に書いてはいけない
  - http://ikawaha.hateblo.jp/entry/2015/10/08/172633


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873118468&linkId=65ce90491608995926ef69b8e0257c2a"></iframe>
