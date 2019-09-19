+++
title= "go mod tidyするとmodule ... found, but does not contain package ...エラーで失敗する"
date= 2019-09-20T08:35:57+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "modules"]
keywords = ["Go", "Golang", "Modules", "tidy", "go mod tidy"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

`go mod tidy`が次のようなのエラーで失敗するとき、エラーを解決するメモ。

```bash
$ go mod tidy
github.com/budougumi0617/til/go/tui/promptui imports
        github.com/manifoldco/promptui imports
        github.com/alecthomas/gometalinter imports
        gopkg.in/alecthomas/kingpin.v3-unstable imports
        github.com/nicksnyder/go-i18n/i18n: module github.com/nicksnyder/go-i18n@latest (v2.0.2+incompatible) found, but does not contain package github.com/nicksnyder/go-i18n/i18n
 
```


<!--more-->

# TL;DR
- `go mod tidy`コマンドが次のエラーで失敗するようになってしまった。
    - `module $MODULE_NAME@latest (v2.0.2+incompatible) found, but does not contain package MODULE_NAME`
- 間接依存しているパッケージ`v2`ディレクトリを作って`go.mod`ファイルを削除したのが原因
- `go.mod`ファイル内で明示的に`v1`系のバージョンを指定すればばよい。

なお、実行環境は以下。

```bash
$ go version
go version go1.13 darwin/amd64
```

# エラー

以下のような`main.go`ファイルと、`go.mod`ファイルを用意する。

```go
package main

import (
        "fmt"

        "github.com/manifoldco/promptui"
)

func main() {
        fmt.Println(promptui.KeyEnter)
}
```
```
module github.com/budougumi0617/til/go/tui/promptui

go 1.13

require (
        github.com/BurntSushi/toml v0.3.1 // indirect
        github.com/alecthomas/units v0.0.0-20190910110746-680d30ca3117 // indirect
        github.com/manifoldco/promptui v0.3.2
        gopkg.in/alecthomas/kingpin.v3-unstable v3.0.0-20180810215634-df19058c872c // indirect
)
 
```

このふたつのファイルは（あまり意味のある結果ではないが、）次のように実行できる。
```bash
$ go run main.go
13
```

ただし、`go mod tidy`をして`go.mod`ファイルと`go.sum`ファイルを整理しようとすると、以下のエラーが発生して`go mod tidy`が完了しない。

```bash
$ go mod tidy
github.com/budougumi0617/til/go/tui/promptui imports
        github.com/manifoldco/promptui imports
        github.com/alecthomas/gometalinter imports
        gopkg.in/alecthomas/kingpin.v3-unstable imports
        github.com/nicksnyder/go-i18n/i18n: module github.com/nicksnyder/go-i18n@latest (v2.0.2+incompatible) found, but does not contain package github.com/nicksnyder/go-i18n/i18n
 
```

# `does not contain package`の原因

エラーの発生原因となる`github.com/nicksnyder/go-i18n`パッケージのコードを確認してみる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/nicksnyder/go-i18n" data-iframely-url="//cdn.iframe.ly/g6KJCX6"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

GitHubで確認すればわかるとおり、2019/09/20現在、上記のパッケージの`lateset`は`v2`ディレクトリ構成になっており、`go.mod`ファイルがなくなっている。

```bash
tree -L 1 $GOPATH/src/github.com/nicksnyder/go-i18n
/Users/budougumi0617/go/src/github.com/nicksnyder/go-i18n
├── CHANGELOG.md
├── LICENSE
├── README.md
├── dev.md
└── v2

1 directory, 4 files
```

そのため、「latestには該当パッケージがない」というエラーメッセージが発生する。


# 明示的に解決に失敗するモジュールのバージョンを指定する
これを解決するには、`v2`以降前のバージョンを明示的に指定してあげればよい。

該当パッケージの`v1`系の最後のバージョンを確認する。
```bash
git tag -l "v1.*"
v1.0.0
v1.1.0
v1.10.0
v1.10.1
v1.2.0
v1.3.0
v1.4.0
v1.5.0
v1.6.0
v1.7.0
v1.8.0
v1.8.1
v1.9.0
```

`v1.10.1`が最新だったので、これを`go.mod`ファイル内で指定する。


```bash
$ go mod edit -require github.com/nicksnyder/go-i18n@v1.10.1
$ cat go.mod
module github.com/budougumi0617/til/go/tui/promptui

go 1.13

require (
        github.com/BurntSushi/toml v0.3.1 // indirect
        github.com/alecthomas/units v0.0.0-20190910110746-680d30ca3117 // indirect
        github.com/manifoldco/promptui v0.3.2
        github.com/nicksnyder/go-i18n v1.10.1
        gopkg.in/alecthomas/kingpin.v3-unstable v3.0.0-20180810215634-df19058c872c // indirect
)
 
```

こうして明示的にバージョンを指定すると、`go mod tidy`コマンドが成功するようになる。

```bash
$ go mod tidy
$ cat go.mod
module github.com/budougumi0617/til/go/tui/promptui

go 1.13

require (
        github.com/BurntSushi/toml v0.3.1 // indirect
        github.com/alecthomas/units v0.0.0-20190910110746-680d30ca3117 // indirect
        github.com/manifoldco/promptui v0.3.2
        github.com/nicksnyder/go-i18n v1.10.1 // indirect
        gopkg.in/alecthomas/kingpin.v3-unstable v3.0.0-20180810215634-df19058c872c // indirect
)
 
```


# 終わりに
いつもエディタで直接編集してしまうが、`go mod edit`コマンドを始めて使った。  
直接編集もそんなに苦労しないので、これは覚えなくてもいいかな…


また、`replace`を使って該当パッケージを`v2`の方へ向けると`go mod tidy`コマンドのせいでビルドが失敗する`go.mod`ファイルができるようになるので、これはissueを立ててみようと思う（不適切な`replace`ならばエラーで終わってほしい）。
