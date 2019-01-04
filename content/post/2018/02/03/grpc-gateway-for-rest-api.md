+++
title= "[Go]gRPC GatewayでgRPCに対するREST APIを自動生成する"
date= 2018-02-03T14:52:19+09:00
draft = false
slug = ""
categories = ["Go","gRPC"]
tags = ["golang", "grpc", "grpc-gateway"]
author = "budougumi0617"
twitterImage = "logos/grpc.png"
keywords = ["Go", "gRPC", "gRPC Gateway"]
+++

gRPCで作ったAPIは通常REST APIからコールできない。  
`grpc-ecosystem/grpc-gateway`を使うとgRPC APIに対するREST APIのリバースプロクシサーバのコードが自動生成出来る。
Protocol BufferにREST用の定義を追加して`grpc-ecosystem/grpc-gateway`を試してみた。


# TL;DR
- `grpc-ecosystem/grpc-gateway`はProtocol BufferからRESTのproxyサーバを自動生成できる
- 認証やロギングを無視すればそのままリバースプロクシサーバを立ち上げられる。
- `curl`コマンドを使ってgRPC APIを実行することができた。

今回作ったサンプルコードは以下にある。

https://github.com/budougumi0617/sandbox-grpc/tree/master/tasklist-gateway

# grpc-ecosystem/grpc-gatewayを使ったREST proxyサーバの構築

Protocol Bufferの定義に`protoc`を使ってGoのコードを作成し、gRPCサーバ/クライアントの作成が一通り出来る環境は出来ているとする。

まずは`grpc-gateway`関連のライブラリを取得する。

```bash
$ go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway
$ go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger
$ go get -u github.com/golang/protobuf/protoc-gen-go
```

## REST API定義をprotoファイルに追加する

以下のページに文法が載っているので、gRPCとRPCの対比を見ながら`.proto`ファイルにオプションをつけていく。

https://cloud.google.com/service-management/reference/rpc/google.api#http

```diff
 // For define null request
 import "google/protobuf/empty.proto";

+// For grpc gateway
+import "google/api/annotations.proto";
+
 service TaskManager {
-  rpc GetTask (GetTaskRequest) returns (Task) {}
-  rpc ListTasks (google.protobuf.Empty) returns (stream Task) {}
+  rpc GetTask (GetTaskRequest) returns (Task) {
+    option (google.api.http) = {
+       get: "/v1/tasklist-gateway/task/{id}" // "id" is defined in GetTaskRequest
+    };
+  }
+  rpc ListTasks (google.protobuf.Empty) returns (stream Task) {
+    option (google.api.http) = {
+       get: "/v1/tasklist-gateway/task"
+    };
+  }
 }
```

## `protoc-gen-grpc-gateway`によるProxyサーバ用APIコードの自動生成


まずは普通の`protoc`コマンドでコンパイルする。`import`を増やしているので、`include`ディレクトリを増やす必要がある。`README`はもう少し`-I`オプションが多いが、自分はこれだけでOKだった。

```
protoc -I. \
  -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
  --go_out=plugins=grpc:. proto/task_list.proto
```

次に、Gateway用のコンパイルを行う。

```bash
protoc -I/usr/local/include -I. -I$GOPATH/src \
  -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
  --grpc-gateway_out=logtostderr=true:. proto/task_list.proto
```

`grpc-gateway`で使えるオプションは2018/01/28の時点では以下の通りだった。`protoc`を実行するときに`--grpc-gateway_out=logtostderr=true`にカンマ区切りでオプションを追加できる。

```
go run $GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway/main.go -h
Usage of /var/folders/sy/ls4cfp216x774g54brzl67yw0000gn/T/go-build442506345/command-line-arguments/_obj/exe/main:
  -allow_delete_body
    	unless set, HTTP DELETE methods may not have a body
  -alsologtostderr
    	log to standard error as well as files
  -import_path string
    	used as the package if no input files declare go_package. If it contains slashes, everything up to the rightmost slash is ignored.
  -import_prefix string
    	prefix to be added to go package paths for imported proto files
  -log_backtrace_at value
    	when logging hits line file:N, emit a stack trace
  -log_dir string
    	If non-empty, write log files in this directory
  -logtostderr
    	log to standard error instead of files
  -request_context
    	determine whether to use http.Request's context or not (default true)
  -stderrthreshold value
    	logs at or above this threshold go to stderr
  -v value
    	log level for V logs
  -vmodule value
    	comma-separated list of pattern=N settings for file-filtered logging
```

例えば、こんな感じ。

```bash
protoc -I/usr/local/include -I. -I$GOPATH/src \
  -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
  --grpc-gateway_out=logtostderr=true,request_context=false,allow_delete_body=true:. \
  proto/task_list.proto
```

`swagger.json`も生成できる。

```bash
protoc -I/usr/local/include -I. -I$GOPATH/src \
  -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
  --swagger_out=logtostderr=true:. proto/task_list.proto
```

`swagger`生成用のオプションは`go run $GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger/main.go -h`で知ることができる。


## サーバコードの実装

もろもろのコードを自動生成できたので、あとはサーバーのコードを実装する。

gRPCサーバは適当なデータを返すように実装しておく。

```go
// sandbox-grpc/tasklist-gateway/server/main.go

// It implements the route guide service whose definition can be found in proto/task_list.proto
package main

import (
	"context"
	"errors"
	"log"
	"net"

	tlpb "github.com/budougumi0617/sandbox-grpc/tasklist-gateway/proto"
	google_protobuf "github.com/golang/protobuf/ptypes/empty"
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
)

const (
	port = ":50051"
)

// server is used to implement sa
type server struct{}

func (s *server) GetTask(ctx context.Context, req *tlpb.GetTaskRequest) (*tlpb.Task, error) {
	log.Println("GetTask in gPRC server")
	var task = &tlpb.Task{
		Id:     10,
		Title:  "Implement gRPC server",
		Detail: "Implement interface",
	}
	if req.Id == task.Id {
		return task, nil
	}
	return nil, errors.New("Not find Task")
}
func (s *server) ListTasks(_ *google_protobuf.Empty, stream tlpb.TaskManager_ListTasksServer) error {
	log.Println("ListTasks in gPRC server")
	tasks := []*tlpb.Task{
		&tlpb.Task{
			Id:     20,
			Title:  "List Tasks 1",
			Detail: "Implement interface",
		},
		&tlpb.Task{
			Id:     21,
			Title:  "List Tasks 2",
			Detail: "Implement interface",
		},
		&tlpb.Task{
			Id:     22,
			Title:  "List Tasks 3",
			Detail: "Implement interface",
		},
	}
	for _, task := range tasks {
		if err := stream.Send(task); err != nil {
			return err
		}
	}
	return nil
}

func main() {
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	tlpb.RegisterTaskManagerServer(s, &server{})
	// Register reflection service on gRPC server.
	reflection.Register(s)
	log.Printf("gRPC Server started: localhost%s\n", port)
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

Proxyサーバはサンプルコードの`import`を変えるだけで良かった。

```go
// sandbox-grpc/tasklist-gateway/gateway/main.go

package main

import (
	"flag"
	"net/http"

	gw "github.com/budougumi0617/sandbox-grpc/tasklist-gateway/proto"
	"github.com/golang/glog"
	"github.com/grpc-ecosystem/grpc-gateway/runtime"
	"golang.org/x/net/context"
	"google.golang.org/grpc"
)

var (
	tasklistEndpoint = flag.String("tasklist_endpoint", "localhost:50051", "endpoint of YourService")
)

func run() error {
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	mux := runtime.NewServeMux()
	opts := []grpc.DialOption{grpc.WithInsecure()}
	err := gw.RegisterTaskManagerHandlerFromEndpoint(ctx, mux, *tasklistEndpoint, opts)
	if err != nil {
		return err
	}

	return http.ListenAndServe(":8080", mux)
}

func main() {
	flag.Parse()
	defer glog.Flush()

	if err := run(); err != nil {
		glog.Fatal(err)
	}
}
```

## 動作確認

それぞれのサーバーを起動する。

```bash
$ go run ./gateway/main.go
```

```bash
$ go run ./server/main.go
```

`curl`コマンドでREST APIを実行してみた。
期待通りにデータが取得できている。

```bash
$ curl -D - -X GET http://localhost:8080/v1/tasklist-gateway/task
HTTP/1.1 200 OK
Content-Type: application/json
Grpc-Metadata-Content-Type: application/grpc
Date: Sun, 28 Jan 2018 04:03:14 GMT
Transfer-Encoding: chunked

{"result":{"id":20,"title":"List Tasks 1","detail":"Implement interface"}}
{"result":{"id":21,"title":"List Tasks 2","detail":"Implement interface"}}
{"result":{"id":22,"title":"List Tasks 3","detail":"Implement interface"}}

$ curl -D - -X GET http://localhost:8080/v1/tasklist-gateway/task/11
HTTP/1.1 500 Internal Server Error
Content-Type: application/json
Trailer: Grpc-Trailer-Content-Type
Date: Sun, 28 Jan 2018 04:03:29 GMT
Transfer-Encoding: chunked

{"error":"Not find Task","code":2}Grpc-Trailer-Content-Type: application/grpc

$ curl -D - -X GET http://localhost:8080/v1/tasklist-gateway/task/10
HTTP/1.1 200 OK
Content-Type: application/json
Grpc-Metadata-Content-Type: application/grpc
Date: Sun, 28 Jan 2018 04:03:33 GMT
Content-Length: 72

{"id":10,"title":"Implement gRPC server","detail":"Implement interface"}%
```

当然だが、gRPC API内に仕込んでおいた実行ログも確認できた。

```bash
go run ./server/main.go
2018/01/28 13:00:48 gRPC Server started: localhost:50051
2018/01/28 13:03:14 ListTasks in gPRC server
2018/01/28 13:03:29 GetTask in gPRC server
2018/01/28 13:03:33 GetTask in gPRC server
```

# 終わりに

gRPCサーバに対応するRESTのリバースプロクシサーバを自動生成した。リバースプロクシサーバに関してはほぼ自分で実装していない。  
（本当は認証とかログのミドルウェア噛ませないといけないというところ？）  
これでgRPCサーバの実装を`curl`コマンドで確認することができた。

# 余談
一連の流れを触っている中で、`GET`メソッドなのにBODYの宣言をつけて`protoc`したら変なエラーメッセージが出ていた。  
IssueをたててPR作ったらマージされた。今まではREADMEなどへのPRしかしてことがなかったので`go get -u`したら結果が自分で変更した内容になったのでちょっと感動した。（今回も文言の修正なので、似たようなものだけど）

- https://github.com/grpc-ecosystem/grpc-gateway/issues/531
- https://github.com/grpc-ecosystem/grpc-gateway/pull/532


# gRPC関連の記事

 - [[Go]gRPCのGo Quick Startをやってみた。](/2018/01/01/hello-grpc-go/)
 - [[Go][gRPC]Server Reflection Tutorialをやってみた](/2018/01/04/server-reflection-tutorial/)
 - [[Go]gRPC Basics: GoからgRPCのストリーミングRPCを理解する](/2018/01/14/grpc-basics-go/)
 - [[Go]gomockを使ったgRPCのテスト](/2018/01/21/try-gomock-on-grpc-go/)


