+++
title= "The Go Playground（play.golang.org）のショートカットキー"
date= 2020-03-13T08:32:11+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "gotips","palyground"]
keywords = ["Go", "golang", "play.golang.org", "Go言語"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

ちょっとした動作確認にThe Go Playgroundを多用する。

- https://play.golang.org/

実は、The Go Playgroundにはショートカットキーが存在するので、マウス操作は必要ない。

<!--more-->

# TL+DR;
- The Go PlaygroundとTour of Goのエディターにはショートカットキーが設定されている。
    - https://tour.golang.org/welcome/1
- `CTRL`キー + `ENTER`キー: `Format`を実行する
    - `Imports`にチェックがあれば、`goimports`も実行する
    - `SHIFT`キー + `ENTER`キー: `Run`を実行する（コードを実行する）
- 実装の詳細は以下のリポジトリで確認できる。
    - https://github.com/golang/playground
    - https://github.com/golang/tools

# The Go Playgroundのショートカットキーについて記載があるところ
ちょっとした動作確認や、サンプルコードを手元（？）で動かしたいとき、The Go Playgroundを開いてそのまま書いている。  
いちいちマウスをにぎるのがめんどくさいので、ショートカットキーを使って実行したり、フォーマットをかけている。  
The Go PlaygroundのショートカットキーはThe Go Playground自体には書いていない。  
実はA Tour of Goの冒頭に書いてあるショートカットキーがThe Go Playgroundでも利用できる。

- https://tour.golang.org/welcome/1

> The tour is interactive. Click the Run button now (or press Shift + Enter) to compile and run the program on a remote server. The result is displayed below the code.

> When you click on Format (shortcut: Ctrl + Enter), the text in the editor is formatted using the gofmt tool. You can switch syntax highlighting on and off by clicking on the syntax button.


# Go Playgroundの実装を確認。
せっかくなので、実装も確認した。
The Go Playgroundは次のリポジトリのコードで実行されている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/playground" data-iframely-url="//cdn.iframe.ly/K2PRvCo"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

このリポジトリを確認すると、`x/tools/godoc/static`パッケージを使って`playground.js`というJavaScriptファイルをロードしている。

- https://github.com/golang/playground/blob/a7e8e783bec503a4489d00a68d6603941455d8c0/server.go#L64-L68

```go
import "golang.org/x/tools/godoc/static"

func (s *server) handlePlaygroundJS(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-type", "text/javascript; charset=utf-8")
	rd := strings.NewReader(static.Files["playground.js"])
	http.ServeContent(w, r, "playground.js", s.modtime, rd)
}
```

`x/tools/godoc/static`パッケージは次のリポジトリにある。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/tools" data-iframely-url="//cdn.iframe.ly/T71XGV2"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

この中の`playground.js`ファイルを確認すると、たしかに`Shift`キーや`Ctrl`キーのイベントを確認していた。  
他にもショートカット設定があるかな？と思ったが、この2つだけのようだ。

- https://github.com/golang/tools/blob/release-branch.go1.14/godoc/static/playground.js#L379-L392

```js
      if (e.keyCode == 13) { // enter
        if (e.shiftKey) { // +shift
          run();
          e.preventDefault();
          return false;
        } if (e.ctrlKey) { // +control
          fmt();
          e.preventDefault();
        } else {
          autoindent(e.target);
        }
      }
```

# その他のThe Go Playground情報
最近のThe Go Playgroundは複数ファイルに分かれたサンプルコードも実行できる。  
なので、パッケージの`import`関係を確認したりすることもできる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://qiita.com/tenntenn/items/4c2d33f795aa6e23e188" data-iframely-url="//cdn.iframe.ly/sCgOC1z?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

また、現在のThe Go Playgroundは標準パッケージ以外もimportできるので、privateなパッケージさえ含まなければ何でも実行できる。

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">The <a href="https://twitter.com/hashtag/golang?src=hash&amp;ref_src=twsrc%5Etfw">#golang</a> playground now supports third-party imports, pulling them in via <a href="https://t.co/IrUZXimjCk">https://t.co/IrUZXimjCk</a> ... <a href="https://t.co/5Ng5JXpggq">https://t.co/5Ng5JXpggq</a> 🎉<br><br>Multi-file support &amp; few other things up next.<br><br>Report bugs at <a href="https://t.co/kZELNa2yzY">https://t.co/kZELNa2yzY</a> or here on the tweeters.</p>&mdash; Brad Fitzpatrick (@bradfitz) <a href="https://twitter.com/bradfitz/status/1128069715455123457?ref_src=twsrc%5Etfw">May 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 終わりに
ショートカットキーについてはThe Go Playground内では言及されていないので、書いた。  
図らずも`static`パッケージの配信方法も見れたので学びがあった。

# 参考
- [The Go Playground](https://play.golang.org/)
- [A Tour of Go](https://tour.golang.org/welcome/1)
- [The Go Playground上で簡単に複数のファイルをシェアする #golang](https://qiita.com/tenntenn/items/4c2d33f795aa6e23e188)
- [x/playground: support third-party imports | golang/go](https://github.com/golang/go/issues/31944)
