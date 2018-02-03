+++
title= "[Go]gRPCのGo Quick Startをやってみた。"
date= 2018-01-01T17:30:50+09:00
draft = false
slug = ""
categories = ["Go","gRPC"]
tags = ["golang", "grpc"]
author = "budougumi0617"
+++

gRPCについて理解を始めるため、gRPCのクイックスタートをやったのでメモしておく。


**Go Quick Start**  
https://grpc.io/docs/quickstart/go.html

ちなみに今回行ったのは、`Go`のクイックスタートだが、公式には以下の言語のクイックスタートが用意されている。

- C++
- Java
- Python
- Go 
- Ruby
- Node.js
- C#
- Objective-C
- Android Java
- PHP

# TL;DR
- `google.golang.org/grpc`リポジトリを取得すればサンプルコードもついてくる
- サンプルの内容は以下
    - 定義済みのプロトコルバッファーを呼ぶコードを動かす
    - プロトコルバッファーに定義を追加して再コンパイルしたAPIを実行する
- `reflection.Register`しているところはとりあえず無視してよいみたい
- Server Reflectionとはなんなのか？別途調べる。


# What is gRPC?

**What is gRPC?**  
https://grpc.io/docs/guides/index.html

翻訳された方がいらっしゃった。

**gRPCとは何か？**  
http://hapoon.hateblo.jp/entry/2017/09/28/171123

「なぜ必要なのか？」はこの辺を読むとよさそう。

**第24回　マイクロサービス・システムにおけるgRPCの役割（前編）**  
https://www.school.ctc-g.co.jp/columns/nakai2/nakai224.html

# gRPCを利用するための準備。
Macの場合、`protobuf`は`brew`経由でインストールできる。

```bash
$ go get -u google.golang.org/grpc
$ brew install protobuf
$ go get -u github.com/golang/protobuf/protoc-gen-go
```

`GOPATH`に`PATH`を通していない場合は、`$ export PATH=$PATH:$GOPATH/bin`もしておく。


`google.golang.org/grpc`の中にサンプルコードがあるのでカレントディレクトリを移動する。サンプルコードの構成は以下。


```
.
├── greeter_client
│   └── main.go
├── greeter_server
│   └── main.go
├── helloworld
│   ├── helloworld.pb.go
│   └── helloworld.proto
└── mock_helloworld
    ├── hw_mock.go
    └── hw_mock_test.go
```

`helloworld.proto`ファイルにプロトコルバッファーと呼ばれるgRPCでやりとりするデータが定義されている。`.proto`ファイルをプロトコルコンパイラー（`protc`）でコンパイルすることで各言語用のgRPCコードを自動生成できる。サンプルコードではすでにコンパイル済みの`helloworld.pb.go`ファイルが同梱されている。

プロトコルバッファーの定義についてもっと知りたいときは以下を参照することで確認できる。

**What is gRPC?**  
https://grpc.io/docs/#what-is-grpc

**gRPC Basics: Go.**  
https://grpc.io/docs/tutorials/basic/go.html

サーバ側を起動しておいてクライアントコードを実行すると、レスポンスが受け取れることを確認できる。

```
$ go run greeter_server/main.go

-----------------

$ go run greeter_client/main.go
2017/12/31 14:11:52 Greeting: Hello world
```

# プロトコルバッファーの更新

`.proto`ファイルに新しいプロトコルをひとつ追加する。

```diff
diff --git a/examples/helloworld/helloworld/helloworld.proto b/examples/helloworld/helloworld/helloworld.proto
index d79a6a0..becd915 100644
--- a/examples/helloworld/helloworld/helloworld.proto
+++ b/examples/helloworld/helloworld/helloworld.proto
@@ -24,6 +24,8 @@ package helloworld;
 service Greeter {
   // Sends a greeting
   rpc SayHello (HelloRequest) returns (HelloReply) {}
+  // Sends another greeting
+  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
 }

 // The request message containing the user's name.
```

その後、`protoc`コマンドで再コンパイルをする。

```bash
$ protoc -I helloworld/ helloworld/helloworld.proto --go_out=plugins=grpc:helloworld
```


そうすると、`.pb.go`ファイルが更新され、新しいコードが自動生成される。

```diff
        SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error)
+       // Sends another greeting
+       SayHelloAgain(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error)
 }

 type greeterClient struct {
@@ -104,11 +106,22 @@ func (c *greeterClient) SayHello(ctx context.Context, in *HelloRequest, opts ...
        return out, nil
 }

+func (c *greeterClient) SayHelloAgain(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error) {
+       out := new(HelloReply)
+       err := grpc.Invoke(ctx, "/helloworld.Greeter/SayHelloAgain", in, out, c.cc, opts...)
+       if err != nil {
+               return nil, err
+       }
+       return out, nil
+}
+
 // Server API for Greeter service
...
```

`greeter_server/main.go`と`greeter_client/main.go`にチュートリアル通りの実装を追加して、実行すると、追加した定義を使った通信ができた。

```bash
$ go run greeter_client/main.go
2017/12/31 16:35:46 Greeting: Hello world
2017/12/31 16:35:46 Greeting: Hello again world
```

# サンプルコードの実装
動作確認だけではなく、実装についても確認した。

# サーバ側の実装
https://github.com/grpc/grpc-go/blob/master/examples/helloworld/greeter_server/main.go

サーバ側の実装は以下のことをしている。


- `GreeterSever`インタフェースを実装した構造体を定義する
- `net.Listen`でTCPプロトコルのリスナーを生成する
- `grpc.NewSever`メソッドで`grpc.Server`オブジェクトを手に入れる
- プロトコルバッファーから自動生成した`RegisterGreeterServer`メソッドで構造体を`grpc.Server`に登録する
- `Serve`で接続の待機を解する。

途中の`reflection.Register`メソッドを使っているのだが、これはServer Reflectionとやらを行うためのものらしいので、今回は触れない。（このHello WorldサンプルがServer Reflectionのチュートリアルも兼ねているからコールされているらしい）

Server Reflectionについては別途調べる。

## クライアント側の実装

https://github.com/grpc/grpc-go/blob/master/examples/helloworld/greeter_client/main.go

クライアント側は通常のAPIクライアントとあまり変わり無さそうな感じの実装。コネクションが`grpc.ClientConn`オブジェクトなのが大きな違い。

- `grpc.Dial`メソッドでコネクションを生成する
- `NewGreeterClient`でクライアントの実装を得る。
- ContextとRequest用の構造体を渡し、クライアントのメソッドをコールする


# クイックスタートを終えて
さっと用意して、プロトコルバッファーを自分でコンパイルして利用するところまで出来てよかった。次はBasic Tutorialwをやってみてもう少し具体的にgRPCについて理解する。

**gRPC Basics: Go.**  
https://grpc.io/docs/tutorials/basic/go.html

また、Server reflectionがわからないままなので調べる。この辺を読めば良いらしい？

**GRPC Server Reflection Protocol**  
https://github.com/grpc/grpc/blob/master/doc/server-reflection.md

**package reflection**  
https://godoc.org/google.golang.org/grpc/reflection

**gRPC Server Reflection Tutorial**  
https://github.com/grpc/grpc-go/blob/master/Documentation/server-reflection-tutorial.md



# 参考URL

**What is gRPC?**  
https://grpc.io/docs/guides/index.html

**gRPCとは何か？**  
http://hapoon.hateblo.jp/entry/2017/09/28/171123

**第24回　マイクロサービス・システムにおけるgRPCの役割（前編）**  
https://www.school.ctc-g.co.jp/columns/nakai2/nakai224.html

**Why do we need to register reflection service on gRPC server**  
https://stackoverflow.com/questions/41424630/why-do-we-need-to-register-reflection-service-on-grpc-server

**gRPC Basics - Go**  
https://grpc.io/docs/tutorials/basic/go.html

**GRPC Server Reflection Protocol**  
https://github.com/grpc/grpc/blob/master/doc/server-reflection.md

**package reflection**  
https://godoc.org/google.golang.org/grpc/reflection

**gRPC Server Reflection Tutorial**  
https://github.com/grpc/grpc-go/blob/master/Documentation/server-reflection-tutorial.md

# gRPC関連の記事

 - [[Go][gRPC]Server Reflection Tutorialをやってみた](/2018/01/04/server-reflection-tutorial/)
 - [[Go]gRPC Basics: GoからgRPCのストリーミングRPCを理解する](/2018/01/14/grpc-basics-go/)
 - [[Go]gomockを使ったgRPCのテスト](/2018/01/21/try-gomock-on-grpc-go/)
 - [[Go]gRPC GatewayでgRPCに対するREST APIを自動生成する](/2018/02/03/grpc-gateway-for-rest-api)

