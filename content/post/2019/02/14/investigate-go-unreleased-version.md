+++
title= "[Go]未リリース版のGoの仕様や実際の動きを確認する #golangjp"
date= 2019-02-14T07:51:53+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["go","gotips", "docker"]
keywords = ["Go"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


未リリース版のGo仕様や実際の動きを確認する。
今回は2019/02/14現在未リリースのGo1.12の仕様を確認した。

<!--more-->

# TL;DR
- 未リリース版の仕様やリリースノートを確認したいときはtip.golang.orgを見る
  - https://tip.golang.org/doc/go1.12
- 未リリース版の挙動を確認したいときはDockerイメージを使うかコードをビルドする
  - https://hub.docker.com/_/golang


# 未リリースのGoの仕様の調べ方
Goの仕様は基本的にgolang.orgで公開されている。当然ここに記載されている仕様はリリース済みの最新版の仕様だ。
2019/02/14現在まだリリースされていないGo1.12の仕様を調査したいときは`tip.golang.org`にアクセスする。
tipのほうならgolang.orgに無い未リリースバージョンの（現時点で予定されている）リリースノートも確認することができる。

- Go 1.12 Release Notes | The Go Programming Language
  - https://tip.golang.org/doc/go1.12
- Packages | The Go Programming Language
  - https://tip.golang.org/pkg/

もろもろの経緯などを調査するときはGitHub上でマイルストーンを確認してもよい。

- Go1.12 | GitHub
  - https://github.com/golang/go/milestone/65

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/golang/go" data-iframely-url="//cdn.iframe.ly/0EU9cFF"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 未リリースのGoの挙動の確認方法

また、実際の挙動を調査したいときはDockerコンテナ上で確認するとローカル環境が汚れなくてよい。

- golang images | docker hub
  - https://hub.docker.com/_/golang

公式イメージは未リリース版のGoのコンテナも公開しているのでそれをローカルで起動する。
2019/02/14現在では`golang:1.12rc1`が公開されている。

```bash
$ docker run --rm -it --name go112 golang:1.12rc1-stretch
root@a646f11d5463:/go# go version
go version go1.12rc1 linux/amd64
```

## Dockerを利用しない場合はローカルでコードをビルドする。

Dockerが使えない場合はコードを直接ビルドしてもよい。Goはソースコードを取得し、`make.bash/make.rc`コマンドを実行するだけだ。
（`all.bash`を使うとテストから始まるので時間がかかる）

- The Go Programming Language
  - https://go.googlesource.com/go

```bash
$ git clone https://go.googlesource.com/go ~/go_master
$ cd ~/go_master/src
$ pwd
/Users/budougumi0617/go_master/src

$ git remote -v
origin  https://go.googlesource.com/go (fetch)
origin  https://go.googlesource.com/go (push)

$ git checkout go1.12rc1

$ ./make.bash
warning: GOPATH set to GOROOT (/Users/budougumi0617/go_master) has no effect
Building Go cmd/dist using /usr/local/Cellar/go/1.11.5/libexec.
Building Go toolchain1 using /usr/local/Cellar/go/1.11.5/libexec.
Building Go bootstrap cmd/go (go_bootstrap) using Go toolchain1.
Building Go toolchain2 using go_bootstrap and Go toolchain1.
warning: GOPATH set to GOROOT (/Users/budougumi0617/go_master) has no effect
Building Go toolchain3 using go_bootstrap and Go toolchain2.
warning: GOPATH set to GOROOT (/Users/budougumi0617/go_master) has no effect
Building packages and commands for darwin/amd64.
warning: GOPATH set to GOROOT (/Users/budougumi0617/go_master) has no effect
---
Installed Go for darwin/amd64 in /Users/budougumi0617/go_master
Installed commands in /Users/budougumi0617/go_master/bin

$ ../bin/go version
warning: GOPATH set to GOROOT (/Users/shimizu-yoichiro/go_master) has no effect
go version go1.12rc1 darwin/amd64

```

Macbook Pro 13inch 2017年版でも5分かからずビルドが完了する。あとは一時的にPATHやGOPATHを変更すればよい。


# 終わりに
ソースコードをビルドしてもいいが、Docker上のほうがいろいろ安心な気がする。（強いGopherは毎日masterをビルドして使っているらしいけれど）
