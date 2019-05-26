+++
title= "[Go] プリミティブ型の配列・スライスを出力するときはその型のverbが使える #golangjp"
date= 2019-05-26T20:32:20+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["golang", "slice", "verb"]
keywords = ["Go", "golang"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++



柴田さん([@yoshiki_shibata](https://twitter.com/yoshiki_shibata))に`fmt.Printf`について教えてもらったのでメモしておく。

<!--more-->

# TL;DR
- 配列やスライスを出力するときは`for`や`range`で出力することが多いだろう
- プリミティブ型の配列・スライスならば、その配列・スライスに対してその型に対応したヴァーブを使うことができる

# Goはverbを使って好きな形式でデータをログや文字列に出力することができる

GoはC言語などの`printf`と同様に`fmt.Printf`関数などを用いてフォーマットした出力を生成できる。
たとえば、文字列は`%s`や`%s`といったヴァーブ(`verb`)が用意されている。

- Printing  | The Go Programming Language
  - https://golang.org/pkg/fmt/#hdr-Printing

https://play.golang.org/p/BNnQmLi3NEM

```go
package main

import (
	"fmt"
)

func main() {
	h, p := "Hello", "playground"
	fmt.Printf("%s, %q\n", h, p) // Hello, "playground"
}
```

ここで、配列やスライスを出力したいときはどうするべきだろうか。
私はいままで`%v`やループを回して出力することが多かった。

https://play.golang.org/p/dCsmGmsWhbv

```go
package main

import (
	"fmt"
)

func main() {
	sa := []string{"a", "bc", "def"}
	fmt.Printf("%v\n", sa)           // [a bc def]
	for _, v := range sa {
		fmt.Printf("%s ", v)           // a bc def
	}
}
```

# プリミティブ型の配列・スライスならば、その配列・スライスに対してその型に対応したヴァーブを使うことができる
柴田さん([@yoshiki_shibata](https://twitter.com/yoshiki_shibata))に実はもっと簡単に出力できることを教えてもらった。
ざっくりいうと、`[]string`型のスライスなどをそのまま`%s`ひとつで出力できるということだ。

https://play.golang.org/p/J5oFCGTuDcQ

```go
package main

import (
	"fmt"
)

func main() {
	sa := []string{"a", "bc", "def"}
	da := []int{1, 2, 3, 4}
	fa := []float32{1.23, 4.5678}
	fmt.Printf("%s\n", sa)           // [a bc def]
	fmt.Printf("%q\n", sa)           // ["a" "bc" "def"]
	fmt.Printf("%d\n", da)           // [1 2 3 4]
	fmt.Printf("%f\n", fa)           // [1.230000 4.567800]
	fmt.Printf("%1.19f\n", fa)       // [1.2300000190734863281 4.5678000450134277344]
}
```

`%1.19f`などの指定もきちんと反映されていることがわかる。
わざわざループを回して表示を整えたりしていたこともあったが、こちらのほうがシンプルに出力できそうだ。

# 最後に
今回の内容は第27回横浜Go読書会で教えてもらった。
勉強会は組み込み系エンジニアの方々が多いので、Go以外にもカーネル周りの苦労話や知識など毎回教えてもらっている。
自分よりスキルのあるみなさんとあれこれ議論しながら課題本を読み進めるので大変勉強になる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://yokohama-go-reading.connpass.com/event/127180/" data-iframely-url="//cdn.iframe.ly/vafxNyK"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 参考
- [Printing  | The Go Programming Language](https://golang.org/pkg/fmt/#hdr-Printing)

