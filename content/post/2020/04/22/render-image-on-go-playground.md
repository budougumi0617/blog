+++
title= "The Go Playgroundの実行結果に画像を出力する"
date= 2020-04-22T00:08:35+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang"]
keywords = ["playground","Go","goalng","Go言語"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Go Playgroundのテンプレートを見ていたら画像を出力できることを発見した。

![実行結果](/2020/04/22_playground.png)

<!--more-->

# TL;DR
- The Go Playgroundにテンプレートが追加された
- テンプレートの中にfaviconを描画するコードがある
- Playground上で`IMAGE:`を先頭に付与してbase64エンコーディングして出力すると画像が表示できる

```go
var buf bytes.Buffer
png.Encode(&buf, img)
fmt.Println("IMAGE:" + base64.StdEncoding.EncodeToString(buf.Bytes()))
```

# The Go Playgroundにテンプレートが追加された
The Go Playgroundはブラウザ上でGoのコードを実行できるオンラインエディタだ。
近年、複数ファイルを含んだサンプルコードを実行できるようになったり、標準パッケージ以外もimportできるようになっている。
そして先日からテンプレートが選択できるようになった。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">Go Playgroundでテンプレートが選べるようになってる！！ <a href="https://twitter.com/hashtag/golang?src=hash&amp;ref_src=twsrc%5Etfw">#golang</a> <a href="https://t.co/LScb4OOfpn">pic.twitter.com/LScb4OOfpn</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1250605368943624192?ref_src=twsrc%5Etfw">April 16, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

テンプレートの内容自体は`github.com/golang/playground/examples`で確認することができる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/playground" data-iframely-url="//cdn.iframe.ly/K2PRvCo"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

このテンプレートの中のひとつにPlayground上で画像を出力するサンプルコードがある。

# The Go Playgroundで出力結果に画像を表示する。
画像を描画するサンプルコードは次のとおり（コードは抜粋したコード）。

- https://github.com/golang/playground/blob/master/examples/image.txt

```go
import (
  "bytes"
  "encoding/base64"
  "fmt"
  "image/png"
)

var buf bytes.Buffer
png.Encode(&buf, img)
fmt.Println("IMAGE:" + base64.StdEncoding.EncodeToString(buf.Bytes()))
```

サンプルコード通りにイメージを標準出力すれば任意のフルカラーの画像をThe Go Playgroundで表示することができる。

ここでは練習問題で多用される（？）マンデルブロ集合を描画してみる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/adonovan/gopl.io" data-iframely-url="//cdn.iframe.ly/ukgf0Qy"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

書籍「プログラミング言語Go」のサンプルコードを使ってThe Go Playground上でマンデルブロ集合の画像を出力するコードは次の通り。

- https://play.golang.org/p/j2Qjgo2F5kR

```go
package main

import (
  "bytes"
  "encoding/base64"
  "fmt"
  "image"
  "image/color"
  "image/png"
  "math/cmplx"
)

func main() {
  displayImage()
}

func displayImage() {
  const (
    xmin, ymin, xmax, ymax = -2, -2, +2, +2
    width, height          = 128, 128
  )

  img := image.NewRGBA(image.Rect(0, 0, width, height))
  for py := 0; py < height; py++ {
    y := float64(py)/height*(ymax-ymin) + ymin
    for px := 0; px < width; px++ {
      x := float64(px)/width*(xmax-xmin) + xmin
      z := complex(x, y)
      // Image point (px, py) represents complex value z.
      img.Set(px, py, mandelbrot(z))
    }
  }
  var buf bytes.Buffer
  err := png.Encode(&buf, img)
  if err != nil {
    panic(err)
  }
  fmt.Println("IMAGE:" + base64.StdEncoding.EncodeToString(buf.Bytes()))
}

func mandelbrot(z complex128) color.Color {
  const iterations = 200
  const contrast = 15

  var v complex128
  for n := uint8(0); n < iterations; n++ {
    v = v*v + z
    if cmplx.Abs(v) > 2 {
      r := 255 - contrast*n
      g := contrast * n
      b := g
      return color.RGBA{r, g, b, 255}
    }
  }
  return color.Black
}
```

上記のコードをThe Go Playground上で実行すると以下のようになる。

![実行結果](/2020/04/22_playground.png)

実行結果としてマンデルブロ集合画像が描画されているのがわかる。

# 終わりに
The Go Playgroundに追加されたテンプレートを眺めていたらこの機能を見つけた。  
追加されたテンプレートの中には複数ファイルの書き方や、CLI上の進捗バー描画などもある。  
実務で使うことはほぼないであろうコードばかりだが、勉強になるので一度全部見ておくと良さそうだ。

# 参考
- https://play.golang.org/
- https://github.com/golang/playground
- https://github.com/adonovan/gopl.io
- https://play.golang.org/p/RNf9hnNvoON


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4621300253&linkId=b0f74778233054f1e700f570fc471f6e"></iframe>