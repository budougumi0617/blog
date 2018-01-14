+++
title= "[Go]gRPC Basics: GoからgRPCのストリーミングRPCを理解する"
date= 2018-01-14T10:47:23+09:00
draft = false
slug = ""
categories = ["Go","gRPC"]
tags = ["golang", "grpc"]
author = "budougumi0617"
+++

gRPC-goのクイックスタート、サーバーレリフレクションを試したので、次はgRPC Basics: Goをやってみた。クイックスタートを終えているので、そちらで学習できている部分（環境構築手順、基本的な概念）には触れない。
クイックスタートでは出てこなかったStreaming RPCについてまとめた。

**gRPC Basics - Go**  
https://grpc.io/docs/tutorials/basic/go.html

なお、上の公式ページの[元MarkDown](https://github.com/grpc/grpc.github.io/blob/master/docs/tutorials/basic/go.md)とgrpc-goリポジトリ内の[同等の内容のMarkDown](https://github.com/grpc/grpc-go/blob/master/examples/gotutorial.md)を比較すると、grpc-goの中の文書の方が新しいので、そちらを読んだ。
Basicsで使うサンプルコードは以下のリポジトリURLにある。

https://github.com/grpc/grpc-go/tree/master/examples/route_guide

# TL;DR
- gRPCはクライアント/サーバーからの単方向ストリーム、双方向ストリームをサポートしている
- protoファイルから自動生成された構造体の`Send`/`Recv`メソッドによってストリームを操作することができる
- ストリームから`io.EOF`が取得されたら送信側からのストリームが終了したことを意味する
- ストリーミングの方式によって、RPC終了時にストリーム操作用の構造体に定義されたメソッドを実行する場合もある

**各ストリーミング形式ごとに定義されたRPC終了時に実行が必要なメソッド**

|ストリーミング方式|Server側|Client側|
|---|---|---|
|server-to-client| - | - |
|client-to-server| `SendAndClose` | `CloseAndRecv` |
|Bidirectional| - | `CloseSend`

# Streaming RPC
gRPCではクライアント/サーバ（あるいは両方）からデータをストリーミング処理で渡すメソッドの定義も行う事ができる。
protoファイルにある該当部分の定義は以下。

https://github.com/grpc/grpc-go/blob/master/examples/route_guide/routeguide/route_guide.proto

```go
  // A server-to-client streaming RPC.
  //
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}

  // A client-to-server streaming RPC.
  //
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}

  // A Bidirectional streaming RPC.
  //
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users).
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
```

protoファイルの定義としてはストリームにしたいメソッドの引数型あるいは戻り値型の前に`stream`と予約語を追加するだけだ。


# gRPC Basics: Go

チュートリアルのサンプルコードのRouteGuideの定義からStreamng RPCの使い方を読み取る。

**gRPC Basics - Go**  
https://grpc.io/docs/tutorials/basic/go.html

# A server-side streaming RPC


サーバからクライアントに対してストリームでデータを渡すRPC。

## サーバサイドの定義
https://github.com/grpc/grpc-go/blob/1cd234627e6f392ade0527d593eb3fe53e832d4a/examples/route_guide/routeguide/route_guide.pb.go#L375

Goの場合は以下のようなメソッドと構造体定義がprotoファイルから生成される。

```go
// プロトコルバッファーの定義は以下。
// rpc ListFeatures(Rectangle) returns (stream Feature) {}
ListFeatures(*Rectangle, RouteGuide_ListFeaturesServer) error

// サーバからクライアントに対してFeature構造体をストリーミングする構造体のインターフェース定義
type RouteGuide_ListFeaturesServer interface {
    Send(*Feature) error
    grpc.ServerStream
}
```

## サーバサイドでストリーミングを送信する実装
https://github.com/grpc/grpc-go/blob/master/examples/gotutorial.md#server-side-streaming-rpc

```go
func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
    for _, feature := range s.savedFeatures {
        if inRange(feature.Location, rect) {
            if err := stream.Send(feature); err != nil {
                return err
            }
        }
    }
    return nil
}
```

ストリーミングは戻り値ではなく、メソッドの引数経由で行うようになる。`RouteGuide_ListFeaturesServer`は`Send(*Feature)`メソッドを持っているので、このメソッドを用いてデータをストリーミングする。
ストリーミングを終了する場合は`return nil`(何らかの異常で終了する場合は当然`return error`)を戻して終了する。

## クライアントサイドの定義
https://github.com/grpc/grpc-go/blob/1cd234627e6f392ade0527d593eb3fe53e832d4a/examples/route_guide/routeguide/route_guide.pb.go#L232

サーバーからストリームを受け取るクライアント側のメソッド定義は以下。

```go
// プロトコルバッファーの定義は以下。
// rpc ListFeatures(Rectangle) returns (stream Feature) {}
ListFeatures(ctx context.Context, in *Rectangle, opts ...grpc.CallOption) (RouteGuide_ListFeaturesClient, error)

// サーバーからストリーミングされるFeature構造体を受け取る構造体のインターフェース定義
type RouteGuide_ListFeaturesClient interface {
    Recv() (*Feature, error)
    grpc.ClientStream
}
```

クライアント側のコードはプロトコルバッファーの定義と似たような形で生成される。戻り値のインターフェースにストリームを受け取る`Recv() (*Feature, error)`メソッドが定義されている。

## クライアントサイドでストリーミングを受け取る実装
https://github.com/grpc/grpc-go/blob/master/examples/gotutorial.md#server-side-streaming-rpc-1

サーバ側の`RouteGuide_ListFeaturesServer.Send`メソッドでストリーミングされたデータを、`RouteGuide_ListFeaturesClient.Recv`メソッドで受け取る。実際にメソッドを利用する際は次のような形になる。



```go
rect := &pb.Rectangle{ ... }  // initialize a pb.Rectangle
stream, err := client.ListFeatures(context.Background(), rect)
if err != nil {
    ...
}
for {
    feature, err := stream.Recv()
    if err == io.EOF { // サーバ側でストリーミングが正常に終了(return nil)された
        break
    }
    if err != nil {
        log.Fatalf("%v.ListFeatures(_) = _, %v", client, err)
    }
    log.Println(feature)
}
```

サーバ側でストリーミングが正常に終了(`return nil`)されたときは、`Recv`メソッドの戻り値として`io.EOF`を受け取ることになる。



# A client-side streaming RPC

クライアントからサーバに対してストリームでデータを渡すRPC。

## サーバサイドの定義
https://github.com/grpc/grpc-go/blob/1cd234627e6f392ade0527d593eb3fe53e832d4a/examples/route_guide/routeguide/route_guide.pb.go#L380

```go
// プロトコルバッファーの定義は以下。
// rpc RecordRoute(stream Point) returns (RouteSummary) {}
RecordRoute(RouteGuide_RecordRouteServer) error

// クライアントからPoint構造体をストリーミングで受け取る構造体のインターフェース定義
type RouteGuide_RecordRouteServer interface {
    SendAndClose(*RouteSummary) error
    Recv() (*Point, error)
    grpc.ServerStream
}
```

クライアントからのストリーミングが終了したときは`RouteGuide_RecordRouteServer.SendAndClose`を呼ぶ必要がある。

## サーバサイドでストリーミングを受け取る実装
https://github.com/grpc/grpc-go/blob/master/examples/gotutorial.md#client-side-streaming-rpc

サーバ側の実装の概要は以下。クライアントからのストリーミングが終了した時、`RouteGuide_RecordRouteServer.Recv`メソッドの戻り値が`io.EOF`となるので、`RouteGuide_RecordRouteServer.SendAndClose`メソッドを実行してメソッドを終了する。

```go
for {
    point, err := stream.Recv() // streamはメソッド引数のRouteGuide_RecordRouteServer
    if err == io.EOF {
        endTime := time.Now()
        return stream.SendAndClose(&pb.RouteSummary{
          // Initialize
        })
    }
    if err != nil {
        return err
    }
    // Do something...
}
```

## クライアントサイドの定義
https://github.com/grpc/grpc-go/blob/1cd234627e6f392ade0527d593eb3fe53e832d4a/examples/route_guide/routeguide/route_guide.pb.go#L237

```go
// プロトコルバッファーの定義は以下。
// rpc RecordRoute(stream Point) returns (RouteSummary) {}
RecordRoute(ctx context.Context, opts ...grpc.CallOption) (RouteGuide_RecordRouteClient, error)

// クライアントからサーバに対してPoint構造体をストリーミングする構造体のインターフェース定義
type RouteGuide_RecordRouteClient interface {
    Send(*Point) error
    CloseAndRecv() (*RouteSummary, error)
    grpc.ClientStream
}
```

server-side streaming RPCのときは`return nil`でストリームを終了していたが、client-sideの場合は`RouteGuide_RecordRouteClient.CloseAndRecv`メソッドでストリームを終了する。

## クライアントでストリーミングを送信する実装
https://github.com/grpc/grpc-go/blob/master/examples/gotutorial.md#client-side-streaming-rpc-1

エラー処理などをほぼ省略した実装は以下の通り。明示的に`CloseAndRecv`メソッドを呼ぶ。

```go
stream, err := client.RecordRoute(context.Background())
for _, point := range points { // pointsはストリーミングするPoint群
    if err := stream.Send(point); err != nil {
        log.Fatalf("%v.Send(%v) = %v", stream, point, err)
    }
}
reply, err := stream.CloseAndRecv()
```

# A bidirectional streaming RPC
サーバ/クライアント双方でストリーミングを行いながらやりとりするRPC

## サーバサイドの定義
https://github.com/grpc/grpc-go/blob/1cd234627e6f392ade0527d593eb3fe53e832d4a/examples/route_guide/routeguide/route_guide.pb.go#L385

```go
// プロトコルバッファーの定義は以下。
// rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
RouteChat(RouteGuide_RouteChatServer) error

// 双方向ストリーミングを行う構造体のインターフェース定義
type RouteGuide_RouteChatServer interface {
    Send(*RouteNote) error
    Recv() (*RouteNote, error)
    grpc.ServerStream
}
```

ひとつの構造体で送信用、受信用のストリームの送受信が行える。


## 双方向ストリーミングを行うサーバ側の実装
https://github.com/grpc/grpc-go/blob/master/examples/gotutorial.md#bidirectional-streaming-rpc

```go
for {
    in, err := stream.Recv() // streamはメソッドの引数のRouteNote
      if err == io.EOF {
          return nil
      }
      if err != nil {
          return err
      }
      key := serialize(in.Location)
                  ... // look for notes to be sent to client
      for _, note := range s.routeNotes[key] {
          if err := stream.Send(note); err != nil {
              return err
          }
      }
}
```

`Recv`メソッドと`Send`メソッドでストリームの送受信を行う点は変わらない。また、`io.EOF`をクライアントから受け取ったあとに`SendAndClose`メソッドの類を実行する必要もない。

## クライアントサイドの定義
https://github.com/grpc/grpc-go/blob/1cd234627e6f392ade0527d593eb3fe53e832d4a/examples/route_guide/routeguide/route_guide.pb.go#L242

```go
// プロトコルバッファーの定義は以下。
// rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
RouteChat(ctx context.Context, opts ...grpc.CallOption) (RouteGuide_RouteChatClient, error)

// 双方向ストリーミングを行う構造体のインターフェース定義
type RouteGuide_RouteChatClient interface {
    Send(*RouteNote) error
    Recv() (*RouteNote, error)
    grpc.ClientStream
}
```

サーバー側同様、送信用、受信用のストリームの操作がひとつのインターフェースにまとめられている。

## 双方向ストリーミングを行うクライアント側の実装
https://github.com/grpc/grpc-go/blob/master/examples/gotutorial.md#bidirectional-streaming-rpc-1

```go
stream, err := client.RouteChat(context.Background())
waitc := make(chan struct{})
go func() {
    for {
        in, err := stream.Recv()
        if err == io.EOF {
          // read done.
          close(waitc)
          return
        }
        if err != nil {
            log.Fatalf("Failed to receive a note : %v", err)
        }
        log.Printf("Got message %s at point(%d, %d)", in.Message, in.Location.Latitude, in.Location.Longitude)
    }
}()
for _, note := range notes {
    if err := stream.Send(note); err != nil {
        log.Fatalf("Failed to send a note: %v", err)
    }
}
stream.CloseSend()
<-waitc
```

基本的な使い方は今まで同様。最後に`CloseSend`メソッドを実行して終わるのがお作法らしい。


# チュートリアルを終えて
protoファイルのひとつの定義からサーバー用、クライアント用のインターフェースや実装が複数自動生成されるため、コードだけ見ても使い方がよくわからなかったのだが、今回のBasicsでだいぶわかった気がする。ただ、このチュートリアルもRPCの引数が単一のstreamだけのサンプルだったりするので、複数引数で定義すると操作方法が変わるかもしれない（あるいは定義できない？）

# gRPC関連の記事

 - [[Go]gRPCのGo Quick Startをやってみた。](/2018/01/01/hello-grpc-go/)
 - [[Go][gRPC]Server Reflection Tutorialをやってみた](/2018/01/04/server-reflection-tutorial/)
