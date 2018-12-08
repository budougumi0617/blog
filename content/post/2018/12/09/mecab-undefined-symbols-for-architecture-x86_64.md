+++
title = "[Go] bluele/mecab-golangを含むコードをビルドすると Undefined symbols for architecture x86_64 と出て失敗する"
date = 2018-12-09T21:30:35+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "mecab"]
+++
他の方のリポジトリにビルドすると表題のエラーが出てしまった。
`bluele/mecab-golang`をimportしている(CGOの設定が必要だった）のが原因だったのでメモしておく。

<!--more-->

# TL;DR
- `bluele/mecab-golang`を`import`するリポジトリの場合は`go get`・`dep`などの`import`解決だけでは不十分
- Macの場合は`brew install mecab`で`mecab`をインストールする
- 環境変数を設定しておく必要がある。`CGO_LDFLAGS`を設定してビルドをすれば問題ない

# Undefined symbols for architecture x86_64
`git clone`、`go get ./...`後のあるリポジトリのtestを実行しようとしたところ、以下のエラーが出てしまった。

```bash
$ go test
# github.com/bluele/mecab-golang
Undefined symbols for architecture x86_64:
  "_mecab_destroy", referenced from:
      __cgo_41e1bb9c719c_Cfunc_mecab_destroy in _x005.o
     (maybe you meant: __cgo_41e1bb9c719c_Cfunc_mecab_destroy)

....

ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
FAIL	github.com/budougumi0617/mecab-project [build failed]
```

どうやら`import`している`bluele/mecab-golang`が`mecab`関係のCGO関数をきちんと呼べていないのが原因のようだった。

# 環境変数を設定してgo getをやり直す
解決方法は`bluele/mecab-golang`のREADMEを見ればすぐにわかった。

- Getting started | bluele/mecab-golang
  - https://github.com/bluele/mecab-golang#getting-started

MacOSの場合はbrew経由で`mecab`と`mecab-config`をインストールできる。

```bash
$ brew install mecab
$ which mecab
/usr/local/bin/mecab
$ which mecab-config
/usr/local/bin/mecab-config
```

インストール後は`bluele/mecab-golang`のREADME通りに環境変数を設定すればよい。
不要かもしれないが、私は`go get`もやり直しておいた。

```bash
$ export CGO_LDFLAGS="`mecab-config --libs`"
$ export CGO_CFLAGS="-I`mecab-config --inc-dir`"
$ go get -u github.com/bluele/mecab-golang
```

とはいえ、`mecab`以外のCGOの設定などもあって`export`で設定はしたくないという人（自分）もいるだろう。
そのような人は以下のようにコマンド実行時に`CGO_LDFLAGS`を渡せばよい。

```bash
$ CGO_LDFLAGS="`mecab-config --libs`" go test
PASS
ok  	github.com/budougumi0617/mecab-project	0.018s
```

ちなみに`mecab-config --libs`が出力する結果は以下だった。

```bash
$ mecab-config --libs
-L/usr/local/Cellar/mecab/0.996/lib -lmecab -lstdc++
```
