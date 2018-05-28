+++
title= "[Go]vim-goとDelveでVim上からGoのデバッグを行う #vim #golang"
date= 2018-05-29T10:00:00+09:00
draft = true
slug = ""
categories = ["go","vim"]
tags = ["golang","vim","delve"]
author = "budougumi0617"
+++

VimでGoを使う人は大半の人が`vim-go`プラグインをインストールしているだろう。

**fatih/vim-go**
https://github.com/fatih/vim-go

今年の3月、ついに`vim-go`が`delve`をサポートした（他のVimプラグインを使えばすでにVimから使えたようだが）。
VimからGoのプログラムをデバッグする方法をまとめる。

https://github.com/fatih/vim-go/blob/master/CHANGELOG.md#117---march-27-2018

```
1.17 - (March 27, 2018)
FEATURES:
Debugger support! Add integrated support for the delve debugger. Use :GoInstallBinaries to install dlv, and see :help go-debug to get started. [GH-1390]
```

なお、`vim-go`はチュートリアルを読めば大半の機能を理解することが出来るが、最近追加された機能まではチュートリアルに載っていない。

**fatih/vim-go-tutorial**
https://github.com/fatih/vim-go-tutorial

# TL;DR
- `vim-go`のデバッグ機能をヘルプからまとめた

上記の通りなので、英語に抵抗がないならば、Vimを開いて`:h go-debug`とすれば良い。
見つからない場合は`vim-go`が古い状態になっている。

https://github.com/fatih/vim-go/blob/7ac1e62fa30d84946e93a3820bb6cf6a431186ed/doc/vim-go.txt#L1872

なお、今回基本的なデバッグ用語(ブレークポイント、ステップインなど)の説明は省略する。

# 事前準備

まずは最新版のvim-goに更新しておく。`vim-go`の`delve`周りに5月に一度更新が入っているので最新版を取得しておいたほうがよい。
dein.vimでpluginをアップデートするときは`vim`を起動して以下のコマンドを実行する。

```vim
:call dein#update()
```

`:h go-debug`と入力して`DEBUGGER`のヘルプが表示されたならば`vim-go`でデバッグが出来るようになっている。

`delve`が動くことも確認しておこう。ターミナルで一通り動くことを確認しておいたほうが良い。
自分の場合、Macで使うとdelveの起動は出来てもデバッグの開始に失敗することがあった。


# Vim上でのデバッグ
今回のデバッグ対象のサンプルとして次のコードを利用した。

https://github.com/budougumi0617/gopl/blob/master/ch01/ex02/echo.go

```
package main

import (
	"fmt"
	"io"
	"os"
	"strconv"
)

var out io.Writer = os.Stdout // modified during testing

func main() {
	for index, arg := range os.Args {
		s := "[" + strconv.Itoa(index) + "] " + arg // Convert integert to string
		fmt.Fprintln(out, s)
	}
}
```

ローカルにある上記のコードをリポジトリルート上をカレントディレクトリとして`vim`で開いた。

```bash
$ cd $GOPATH/src/github.com/budougumi0617/gopl/
$ vim ch01/ex02/echo.go
```
