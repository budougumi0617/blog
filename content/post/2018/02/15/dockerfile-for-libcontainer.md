+++
title= "[Goならわかるシステムプログラミング]libcontainerで実装したLinuxコンテナを起動するためのDockerfile"
date= 2018-02-15T08:33:14+09:00
draft = false
slug = ""
categories = ["go"]
tags = ["golang","docker","libcontainer"]
author = "budougumi0617"
+++

**Goならわかるシステムプログラミング ― 第20回 Go言語とコンテナ**  
http://ascii.jp/elem/000/001/502/1502967/


Goならわかるシステムプログラミングの中で、GoとlibcontainerライブラリでLinuxコンテナを実装する回があるのだが、
実装したコンテナはLinux上でしか`go build`できないし、起動もできなかった。VirtualBoxなどでも良いが、Dockerで環境を用意するようにした。

今回作成したDockerfileなどは以下にある。

**budougumi0617/gsp/ch17/container**  
https://github.com/budougumi0617/gsp/tree/master/ch17/container


# TL;DR
- 「Goならわかるシステムプログラミング Go言語とコンテナ」のサンプルコードをmacOSやWindowsでも動かせるようにDockerfileを作成した
  - https://github.com/budougumi0617/gsp/blob/master/ch17/container/Dockerfile
- 起動したAlpineコンテナの中でlibcontainerで実装した自作（写経）コンテナを起動できた
- `docker run`するときに`--privileged`オプションをつけておくこと

# 背景

[Goならわかるシステムプログラミング](https://amazon.jp/dp/4908686033)の「第17章 Go言語とコンテナ」では
libcontainerライブラリを使ってGoでコンテナを実装し起動する。

**Goならわかるシステムプログラミング**  
https://amazon.jp/dp/4908686033

**opencontainers/runc/libcontainer**  
https://github.com/opencontainers/runc/tree/master/libcontainer

Web上の連載で言うと第20回になる。

**Goならわかるシステムプログラミング ― 第20回 Go言語とコンテナ**  
http://ascii.jp/elem/000/001/502/1502967/

ただし、Linux上でしか起動しないコンテナなのでmacOS(Windows)上では`go build`すらできない。


# 本中のサンプルコードを実装する
Dockerfileをそのまま利用する場合は本（連載）の中のコードを`_main.go`という名前で写経する。

https://github.com/budougumi0617/gsp/blob/master/ch17/container/_main.go


Linux上でそれなりに準備しておかないとビルドに失敗するので、`_`付きの名前で後で作成するコンテナ内でしかビルドしないようにしておいたほうが良い。  
`build constraints`で`// +build linux`もつけているのだが、自分の`Circle CI`のビルドだとLinux上でのビルドのはずなのに失敗した。

# Dockerfileを作成する
Dockerfileの内容は以下。alpineベースのコンテナに`go build`するときに要求されるライブラリを入れた後、ビルドするだけの単純な構成。

```dockerfile
FROM golang:1.9.4-alpine3.7

RUN apk add --update openssl-dev pcre-dev git gcc musl-dev linux-headers sudo

WORKDIR /go/src/github.com/budougumi0617/gsp/ch17/container/
ADD _main.go ./main.go
ADD rootfs ./rootfs/
RUN go get -v .
RUN go build main.go
CMD echo "test"
```

`alpine`コンテナだとインクルードファイルなどが入っていないのでもろもろ`apk add`しておく。  
また、コンテナに入って自作コンテナでいろいろコマンドを実行するのが目的のためエントリはただの`echo`にしてある。

本（連載）に記載されている`rootfs`ディレクトリにalpineコンテナから抽出したもろもろのファイルを入れる作業は
Dockerfile外で行う必要があるので注意。

```bash
$ docker pull alpine
$ docker run --name alpine alpine
$ docker export alpine > alpine.tar
$ docker rm alpine
$ mkdir rootfs
$ tar -C rootfs -xvf alpine.tar
```

これで自作（写経）コンテナを実行してみる準備ができた。

# docker runするときは--privilegedオプションをつけること
`docker build`の中で`go build`するので、あとはコンテナに入って実際に起動してみる。  
ここでオプションをつけずにそのまま`docker run`をすると以下のようなエラーで終了してしまう。

```bash
$ docker build -t build-container .
$ docker run -it build-container /bin/ash
/go/src/github.com/budougumi0617/gsp/ch17/container # sudo ./main
WARN[0000] signal: killed
2018/02/14 00:43:37 container_linux.go:348: starting container process caused "process_linux.go:279: applying cgroup configuration for process caused \"mkdir /sys/fs/cgroup/cpuset/system: read-only file system\""
```

`syscall`する権限が不足しているのが原因らしいので、`--privileged`をつけて`docker run`する。

```bash
$ docker build -t build-container . --no-cache=true
$ docker run --privileged -it build-container /bin/ash
/go/src/github.com/budougumi0617/gsp/ch17/container # ./main
/bin/sh: can't access tty; job control turned off
/ # /bin/hostname
testing
/ # exit
/go/src/github.com/budougumi0617/gsp/ch17/container # exit
```

ちょっとTTYの設定は出来ていないが、ちゃんと実行することができた。

# おわりに
alpine linuxベースのコンテナを実行させるためにalpine linuxコンテナを作るというやや本末転倒気味のことしているのはご愛嬌。  
頑張れば作業工程の`alpine.tar`を作る工程は削除できそうなのだが、目的は達成したのでここまでにしておく。  
去年の今頃Dockerの本一冊読んだのだが、Dockerfileの書き方をすっかり忘れていた。やはり日頃の継続と反復が大事である。

# 参考
**Goならわかるシステムプログラミング**  
https://amazon.jp/dp/4908686033

**Goならわかるシステムプログラミング ― 第20回 Go言語とコンテナ**  
http://ascii.jp/elem/000/001/502/1502967/

**budougumi0617/gsp/ch17/container**  
https://github.com/budougumi0617/gsp/tree/master/ch17/container

**opencontainers/runc/libcontainer**  
https://github.com/opencontainers/runc/tree/master/libcontainer

**GoのBuild Constraintsに関するメモ**  
https://qiita.com/hnw/items/7dd4d10e837158d6156a

**Docker privileged オプションについて**  
https://qiita.com/muddydixon/items/d2982ab0846002bf3ea8

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4908686033&linkId=5b2bafbf1cb17ffa1d3a6d19e3e12e23"></iframe>

