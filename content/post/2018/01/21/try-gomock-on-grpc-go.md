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

# モック対象のgRPCコード
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

このProtocol BuffersからGoで利用するgRPCのAPIコードを自動生成する。
```bash
$ protoc --go_out=plugins=grpc:. proto/task_list.proto
```

生成されたクライアント用の`interface`は以下。

```go
// Client API for TaskManager service

type TaskManagerClient interface {
^   GetTask(ctx context.Context, in *GetTaskRequest, opts ...grpc.CallOption) (*Task, error)
^   ListTasks(ctx context.Context, in *google_protobuf.Empty, opts ...grpc.CallOption) (TaskManager_ListTasksClient, error)
}

type TaskManager_ListTasksClient interface {
^   Recv() (*Task, error)
^   grpc.ClientStream
}
```


# gRPC関連の記事

 - [[Go]gRPCのGo Quick Startをやってみた。](/2018/01/01/hello-grpc-go/)
 - [[Go][gRPC]Server Reflection Tutorialをやってみた](/2018/01/04/server-reflection-tutorial/)
 - [[Go]gRPC Basics: GoからgRPCのストリーミングRPCを理解する](/2018/01/14/grpc-basics-go/)
