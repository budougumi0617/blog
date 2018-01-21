+++
title= "[Go][gRPC]Server Reflection Tutorialをやってみた"
date= 2018-01-04T08:54:30+09:00
draft = false
slug = ""
categories = ["Go","gRPC"]
tags = ["golang", "grpc"]
author = "budougumi0617"
+++


[前回](/2018/01/01/hello-grpc-go/)のgRPCのクイックスタートをやったときにサーバーリフレクションのことがよくわからなかった。サーバーリフレクションにもチュートリアルがあったのでやってみた。

# TL;DR
- サーバーリフレクションを使うと実行中のサーバからプロトコルバッファーの定義を取得したり、実行出来るようになる。
- Goで利用するには、`grpc.NewServer()`のあとで通常のプロトコルバッファーの`Register`をした後`reflection.Register`メソッドを呼ぶだけ
- [チュートリアル](https://github.com/grpc/grpc-go/blob/master/Documentation/server-reflection-tutorial.md)の手順通りに行うとc++のクライアントからサーバのプロトコルバッファーを確認することができる。

# Server Reflectionとは
**GRPC Server Reflection Protocol**  
https://github.com/grpc/grpc/blob/master/doc/server-reflection.md

サーバーリフレクションはクライアントのコマンドラインからgRPCの定義の取得あるいは実行を可能にするgRPCサーバの拡張機能。サーバーリフレクションによって`curl`ライクなことがgRPCサーバでも実現できる。


# Server Reflection Tutorial

**gRPC Server Reflection Tutorial**  
https://github.com/grpc/grpc-go/blob/master/Documentation/server-reflection-tutorial.md

## grpc_cliのビルド
サーバ側の`Server Reflection`情報を確認するためのクライアントツールを用意する。実装は`c++`のみ提供されているようだ。`grpc/grpc`リポジトリを取得してクライアントツールをビルドする。

```bash
$ git clone https://github.com/grpc/grpc
$ cd grpc
$ make grpc_cli
```

サーバーリフレクションのチュートリアルにはこれしか書かれていないが、依存ライブラリがあるのでそちらのインストールもする。  
自分の場合はMacOSなので以下を行ったあとビルドできた。


```bash
$ git submodule update --init
$ brew install autoconf automake libtool shtool
$ brew install gflags
```

もし動かなかったり、他のOS環境の場合は、`make run_dep_checks`コマンドでビルド環境が整っているか確認する。または、以下のページで他に依存しているものが無いか確認する。

https://github.com/grpc/grpc/blob/master/INSTALL.md

## サーバーに対してgprc_cliを使ってみる

`grpc_cli`のビルドが終わったり`google.golang.org/grpc`リポジトリで、サンプルのhelloworldサーバーを立ち上げておく。

```bash
$ go run examples/helloworld/greeter_server/main.go
```

helloworldサーバーは`git clone`した直後でも、`reflection.Register`が既に呼ばれている。（以下の実行結果はクイックスタートのチュートリアルを行ったあとのサーバのコードなので、`SayHelloAgain`メソッドが追加されている)

```go
s := grpc.NewServer()
pb.RegisterGreeterServer(s, &server{})
// Register reflection service on gRPC server.
reflection.Register(s)
```

あとは立ち上げたサーバーに向けて`grpc_cli ls`コマンドを実行すると、プロトコルバッファーが取得できる。

```bash
./bins/opt/grpc_cli ls localhost:50051 -l
filename: helloworld.proto
package: helloworld;
service Greeter {
  rpc SayHello(helloworld.HelloRequest) returns (helloworld.HelloReply) {}
  rpc SayHelloAgain(helloworld.HelloRequest) returns (helloworld.HelloReply) {}
}

filename: grpc_reflection_v1alpha/reflection.proto
package: grpc.reflection.v1alpha;
service ServerReflection {
  rpc ServerReflectionInfo(stream grpc.reflection.v1alpha.ServerReflectionRequest) returns (stream grpc.reflection.v1alpha.ServerReflectionResponse) {}
}
```



`call`オプションでメソッドを実行させることもできた。

```
$ ./bins/opt/grpc_cli call localhost:50051 SayHelloAgain "name: 'gRPC CLI'"
connecting to localhost:50051
message: "Hello again gRPC CLI"

Rpc succeeded with OK status
```

## チュートリアルを終えて

reflectionパッケージにはRegisterメソッドしかない。

https://godoc.org/google.golang.org/grpc/reflection

現状`Go`のコードで直接リフレクションデータを取得できるようにはなっていないっぽいが、`grpc_reflection_v1alpha`にリフレクション結果の構造体定義がもろもろされている（`grpc_cli ls`コマンドの結果にも含まれている）。後々`Go`のコードからでも扱えるようになるのかな？

https://godoc.org/google.golang.org/grpc/reflection/grpc_reflection_v1alpha

`grpc_cli`のコマンドのusageでオプションを見ると、もろもろの認証情報も付加できるようなので、本番用に構築したgRPCサーバでも利用できそう。

```bash
./bins/opt/grpc_cli ls

Wrong number of arguments for ls

List services
  grpc_cli ls <address> [<service>[/<method>]]
    <address>                ; host:port
    <service>                ; Exported service name
    <method>                 ; Method name
    --l                      ; Use a long listing format
    --outfile                ; Output filename (defaults to stdout)
    --enable_ssl             ; Set whether to use tls
    --use_auth               ; Set whether to create default google credentials
    --access_token           ; Set the access token in metadata, overrides --use_auth
```

# 参考URL
**GRPC Server Reflection Protocol**  
https://github.com/grpc/grpc/blob/master/doc/server-reflection.md

**gRPC Server Reflection Tutorial**  
https://github.com/grpc/grpc-go/blob/master/Documentation/server-reflection-tutorial.md

**GoDoc:google.golang.org/grpc/reflection**  
https://godoc.org/google.golang.org/grpc/reflection

**GoDoc:google.golang.org/grpc/reflection/grpc_reflection_v1alpha**  
https://godoc.org/google.golang.org/grpc/reflection/grpc_reflection_v1alpha

**gRPC Service discovery with Server Reflection and gRPC CLI in Go**  
https://www.goheroe.org/2017/08/19/grpc-service-discovery-with-server-reflection-and-grpc-cli-in-go/


# gRPC関連の記事
 - [[Go]gRPCのGo Quick Startをやってみた。](/2018/01/01/hello-grpc-go/)
 - [[Go]gRPC Basics: GoからgRPCのストリーミングRPCを理解する](/2018/01/14/grpc-basics-go/)
 - [[Go]gomockを使ったgRPCのテスト](/2018/01/21/try-gomock-on-grpc-go/)
