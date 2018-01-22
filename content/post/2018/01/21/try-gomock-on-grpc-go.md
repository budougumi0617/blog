+++
title= "[Go]gomockを使ったgRPCのテスト"
date= 2018-01-21T19:21:28+09:00
draft = false
slug = ""
categories = ["Go","gRPC"]
tags = ["golang", "grpc", "test", "gomock"]
author = "budougumi0617"
+++

gRPCの勉強というより、gomockの勉強と言ったほうが良いかもしれないが、
`protoc`コマンドで生成した`gRPC`コードをモックしてテストする方法を確かめた。
せっかくなので今回は自分でProtocol Bufferの定義からやった。

**Mocking Service for gRPC**  
https://github.com/grpc/grpc-go/blob/master/Documentation/gomock-example.md

# TL;DR
- gRPCで自動生成したGoのAPIのMockを作成する。
- 今回はチュートリアル同様クライアントコードのみ。
- チュートリアルコードと同じパッケージ構成だとちょっとハマった

作成したコードは以下のリポジトリにある。

https://github.com/budougumi0617/sandbox-grpc/tree/master/tasklist

本文中で言及していないが、サンプルコードのディレクトリ構成は以下のようになっている。
```bash
tasklist
├── client
│   └── client_test.go
├── mockproto
│   └── tl_mock.go
└── proto
    ├── task_list.pb.go
    └── task_list.proto
```

# 事前準備
gRPC自体の環境構築は以下を参照のこと。

 - [[Go]gRPCのGo Quick Startをやってみた。](/2018/01/01/hello-grpc-go/)

また、`gomock`を利用する準備をする。

```bash
$ go get github.com/golang/mock/gomock
$ go get github.com/golang/mock/mockgen
```

# モック対象のgRPCコードの用意
まずProtocol Buffersを定義し、モックするgRPCのインターフェースを生成する。

Protocol Buffersの定義は以下

```
syntax = "proto3";

package tasklist;

import "google/protobuf/empty.proto";

service TaskManager {
  rpc GetTask (GetTaskRequest) returns (Task) {}
  rpc ListTasks (google.protobuf.Empty) returns (stream Task) {}
}

message Task {
  int32 id = 1;
  string title = 2;
  string detail = 3;
}

message GetTaskRequest {
  int32 id = 1;
}
```

通常のUnary RPCとストリームを利用したServer-side streamng RPCのメソッドを用意した。

このProtocol BuffersからGoで利用するgRPCのAPIコードを自動生成し、これをモックするテストを書く。

まず、`protoc`によってAPIを自動生成する。
```bash
$ protoc --go_out=plugins=grpc:. proto/task_list.proto
```

クライアント用の`interface`が以下のように自動生成される。

```go
// Client API for TaskManager service

type TaskManagerClient interface {
    GetTask(ctx context.Context, in *GetTaskRequest, opts ...grpc.CallOption) (*Task, error)
    ListTasks(ctx context.Context, in *google_protobuf.Empty, opts ...grpc.CallOption) (TaskManager_ListTasksClient, error)
}

type TaskManager_ListTasksClient interface {
    Recv() (*Task, error)
    grpc.ClientStream
}
```

このインターフェースのモックを`mockgen`コマンドで自動生成する。
２つ以上のインターフェースを指定する時は、`,`のあとにスペースを含めてはいけないようだ。

```
$ mkdir mock_tasklist
$ mockgen github.com/budougumi0617/sandbox-grpc/tasklist/proto TaskManagerClient,TaskManager_ListTasksClient > mockproto/tl_mock.go
```

これでモックコードが作成出来たので、テストコードを作る。
ちなみに、`grpc-go`リポジトリに入ってる[サンプル](https://github.com/grpc/grpc-go/tree/master/examples/route_guide)だと`mock_routeguide`ディレクトリにテストコードも入っているが、`import`がちゃんと解決できなくなる気がするので、同じ構成にしないほうがよい。

# gomockを利用したテスト
作成したテストコードは以下。

[https://github.com/budougumi0617/sandbox-grpc/blob/master/tasklist/client/client_test.go](https://github.com/budougumi0617/sandbox-grpc/blob/bb92fd1fb4df0dfa64de0e4b4ae9fec8997c4e76/tasklist/client/client_test.go)

## Unary RPCのモックを利用したテスト

`GetTask`メソッドのモックを利用するテストを書く。
```go
type TaskManagerClient interface {
  GetTask(ctx context.Context, in *GetTaskRequest, opts ...grpc.CallOption) (*Task, error)
}
```

`Id = 1`が指定されたときだけ、、`Task`オブジェクトを戻すモックを設定する。
```go
import (
    // ...
    tlmock "github.com/budougumi0617/sandbox-grpc/tasklist/mockproto"
    tlpb "github.com/budougumi0617/sandbox-grpc/tasklist/proto"
)

func TestGetTask(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    mockTaskManagerClient := tlmock.NewMockTaskManagerClient(ctrl)
    req := &tlpb.GetTaskRequest{Id: 1}
    mockTaskManagerClient.EXPECT().GetTask(
        gomock.Any(),
        req,
    ).Return(task, nil)
    testGetTask(t, mockTaskManagerClient)
}

func testGetTask(t *testing.T, client tlpb.TaskManagerClient) {
    t.Helper()
    resp, err := client.GetTask(context.Background(), &tlpb.GetTaskRequest{Id: 2})
    if err != nil || resp.Title != task.Title {
        t.Errorf("mocking failed")
    }
    t.Log("Reply : ", resp.Title)
}
```

転記したコードでは`testGetTask`メソッドの中で`Id = 1`でない`GetTaskRequest`でモックコードを呼び出している。

```go
resp, err := client.GetTask(context.Background(), &tlpb.GetTaskRequest{Id: 2})
```

```bash
$ go test . -v
=== RUN   TestGetTask
--- FAIL: TestGetTask (0.00s)
    controller.go:150: Unexpected call to *mock_proto.MockTaskManagerClient.GetTask([context.Background id:2 ]) at /Users/budougumi0617/go/src/github.com/budougumi0617/sandbox-grpc/tasklist/mockproto/tl_mock.go:46 because:
    Expected call at /Users/budougumi0617/go/src/github.com/budougumi0617/sandbox-grpc/tasklist/tasklist_client_test.go:45 doesn't match the argument at index 1.
    Got: id:2
    Want: is equal to id:1
  asm_amd64.s:509: missing call(s) to *mock_proto.MockTaskManagerClient.GetTask(is anything, is equal to id:1 ) /Users/budougumi0617/go/src/github.com/budougumi0617/sandbox-grpc/tasklist/tasklist_client_test.go:45
  asm_amd64.s:509: aborting test due to missing call(s)
FAIL
exit status 1
FAIL  github.com/budougumi0617/sandbox-grpc/tasklist  0.011s
```

ちゃんと失敗した。

## Streaming RPCのモックを使ったテスト
ストリーミングの場合は`ListTasks`メソッドと`TaskManager_ListTasksClient`のモックを利用する。

```go
type TaskManagerClient interface {
  ListTasks(ctx context.Context, in *google_protobuf.Empty, opts ...grpc.CallOption) (TaskManager_ListTasksClient, error)
}
type TaskManager_ListTasksClient interface {
    Recv() (*Task, error)
    grpc.ClientStream
}
```

今回は２つ`Task`オブジェクトを返して終了するストリームをモックに設定した。

```go
import (
    // ...
    tlmock "github.com/budougumi0617/sandbox-grpc/tasklist/mockproto"
    tlpb "github.com/budougumi0617/sandbox-grpc/tasklist/proto"
)

func TestListTasks(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    // Create mock for the stream returned by ListTasks
    stream := tlmock.NewMockTaskManager_ListTasksClient(ctrl)
    stream.EXPECT().Recv().Return(&tlpb.Task{
        Id:     1,
        Title:  "first Task",
        Detail: "fist Detail",
    }, nil)
    stream.EXPECT().Recv().Return(&tlpb.Task{
        Id:     2,
        Title:  "second Task",
        Detail: "second Detail",
    }, nil)
    // io.EOFを戻すと終了したことになる
    stream.EXPECT().Recv().Return(nil, io.EOF)
    // Create mock for the client interface.
    mockclient := tlmock.NewMockTaskManagerClient(ctrl)
    mockclient.EXPECT().ListTasks(
        gomock.Any(), // 引数は無視する（任意にしておく）
        gomock.Any(),
    ).Return(stream, nil)
    testListTasks(t, mockclient)
}

func testListTasks(t *testing.T, client tlpb.TaskManagerClient) {
    t.Helper()
    // ストリームを取得する
    ltc, _ := client.ListTasks(context.Background(), nil)
    // ストリームからオブジェクトを取得する
    first, err := ltc.Recv()
    if err != nil || first.Title != "first Task" {
        t.Errorf("Unexpected task at first response")
    }
    second, err := ltc.Recv()
    if err != nil || second.Title != "second Task" {
        t.Errorf("Unexpected task at second response")
    }
    // 最後はio.EOFで終了する
    _, eof := ltc.Recv()
    if eof != io.EOF {
        t.Error(eof)
    }
}
```

Streaming RPCを使う時、モックの設定を2つすることだけ覚えていれば、あとは問題なさそうだ。PASSするだけなので、結果は省略。


# 終わりに
標準パッケージのみでテストすべきなのかなとも思いつつ、今回はgomockを利用したテストを書いてみた。
自分でProtocol Buffersの定義をしたのも、gomockを使ったのも始めてだったので、２つとも経験できてよかった。

# 参考
**Mocking Service for gRPC**  
https://github.com/grpc/grpc-go/blob/master/Documentation/gomock-example.md

**GoDoc : gomock**  
https://godoc.org/github.com/golang/mock/gomock

**Go Mockでインタフェースのモックを作ってテストする #golang**  
https://qiita.com/tenntenn/items/24fc34ec0c31f6474e6d

# gRPC関連の記事

 - [[Go]gRPCのGo Quick Startをやってみた。](/2018/01/01/hello-grpc-go/)
 - [[Go][gRPC]Server Reflection Tutorialをやってみた](/2018/01/04/server-reflection-tutorial/)
 - [[Go]gRPC Basics: GoからgRPCのストリーミングRPCを理解する](/2018/01/14/grpc-basics-go/)
