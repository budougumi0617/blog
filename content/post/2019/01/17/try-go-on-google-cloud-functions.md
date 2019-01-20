+++
title= "BetaリリースされたGoのGoogle Cloud Functionsを試す #gcp #golangjp"
date= 2019-01-17T10:21:04+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["gcp", "go"]
tags = ["golang", "gcp","functions"]
keywords = ["Go", "golang", "gcp", "Google Cloud Functions"]
twitterImage = "2019/01/gofunctions.png"
+++
Google Cloud FunctionsでついにGoがサポートされた（まだベータリリースだが）。  
さっそくさわってみたメモ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://cloud.google.com/blog/products/application-development/cloud-functions-go-1-11-is-now-a-supported-language/" data-iframely-url="//cdn.iframe.ly/mGNAEkA"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- Go1.11でCloud Functionsを書くことができるようになった
- Cloud Functionsの定義は主に2種類
  - `gcloud functions deploy ${ファンクション名} --runtime go111 --entry-point ${関数名} ${Functions実行時のトリガー}`でデプロイできる
- HTTP Functions
  - `http.HandlerFunc`インターフェースを満たす関数
  - Functions以外の設定をせずに外部公開されてエンドポイントが与えられる
- Background Functions
  - 第一引数が`context.Context`、第二引数が任意の`struct`な関数
  - 特定のGCP上のイベントをトリガーにして起動する
  - イベントの情報は第二引数の構造体に含まれるが、公式から構造体はまだ公開されていない
      - 公式ガイドのサンプルコードを見るとよい



# GoのFunctionsを作成する
まず以下のドキュメントを参考にGoのGunctions用の関数を定義する。

- The Go Runtime | Documentation | Cloud Functions
  - https://cloud.google.com/functions/docs/concepts/go-runtime?hl=en

## HTTPをトリガーにして実行するHTTP Functionsの場合
HTTPアクセスをトリガーにして実行されるFunctionsを定義する場合は、`http.HandlerFunc`インターフェースを満たす関数を用意しておけば良い。

- https://golang.org/pkg/net/http/#HandleFunc

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```

## イベントをトリガーにして実行するBackground Functionsの場合
Cloud Storageなどで発生したイベントをトリガーに起動するFunctionsを定義することもできる。

```go
type GCSEvent struct {
  // 省略
}

// HelloGCSInfo はGCSのイベントをトリガーに起動するBackground Functions用の関数
func HelloGCSInfo(ctx context.Context, e GCSEvent) error {
  // Do anything...
}
```

---

今回はリリース記事に記載されていたHTTP Functionsを少し改良した以下のソースコードを利用する。**パッケージ名は`main`でなければ良い。**

```go
package functions

import (
        "fmt"
        "log"
        "net/http"
)

// F is sample functions
func F(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "text/plain; charset=utf-8")
        fmt.Println("Hello by fmt pkg")
        log.Println("Hello by log pkg")
        w.Write([]byte(r.Header.Get("X-Forwarded-For")))
}
```

# Functionsをデプロイする

まずは`gcloud`コマンドで選択中のプロジェクトのCloud FunctionsのAPIを有効化しておく。

- Cloud Functions API | IAM Library
  - https://console.cloud.google.com/apis/library/cloudfunctions.googleapis.com

また、ローカルの`gcloud`コマンドも以下のコマンドで最新にしておく。
なお、最新にしても`gcloud functions deploy --help`の`runtime`オプションに`go111`の記述はなかったが、ちゃんと使うことができた。

```
$ gcloud components update
$ gcloud components install beta
```

あとは、作成したローカルのGoの関数定義があるディレクトリで`gcloud functions deploy`コマンドを実行すれば良い。
以下のコマンド実行結果はHTTPをトリガーとする`hello`という名前のCloud Functionsを`F`という名前のGoの関数をエントリポイントとして公開したときのログ。
デプロイ完了までそこそこかかるので少し待つ必要があった。

```bash`
$ gcloud functions deploy hello --runtime go111 --entry-point F --trigger-http --region asia-northeast1
Deploying function (may take a while - up to 2 minutes)...done.
availableMemoryMb: 256
entryPoint: F
httpsTrigger:
  url: https://asia-northeast1-MYPROJECT_NAME.cloudfunctions.net/hello
labels:
  deployment-tool: cli-gcloud
name: projects/MYPROJECT_NAME/locations/asia-northeast1/functions/hello
runtime: go111
serviceAccountEmail: MUPROJECT_NAME@appspot.gserviceaccount.com
sourceUploadUrl: https://storage.googleapis.com/g....
status: ACTIVE
timeout: 60s
updateTime: '2019-01-16T23:11:50Z'
versionId: '1'
```

デプロイ後の実行結果ログの`httpsTrigger`という項目がHTTPのエントリポイントなので、そのアドレスに対して`curl`コマンドを実行してみる。


```bash`
$ curl -i https://asia-northeast1-MYPROJECT_NAME.cloudfunctions.net/hello
HTTP/2 200
content-type: text/plain; charset=utf-8
function-execution-id: pzhl9rayoaq6
x-cloud-trace-context: f7bc8c67d13641651434d8b3dc2b2444;o=1
date: Wed, 16 Jan 2019 23:14:58 GMT
server: Google Frontend
content-length: 14
alt-svc: quic=":443"; ma=2592000; v="44,43,39,35"

XXX.XXX.XXX.82%
```

先ほどのGoの関数がちゃんと実行された！！




# Cloud Functions実行時のログについて
とくに記載はなかったような気がするが、標準出力に出ていればStackdriverで確認できるようだ。
`fmt`と`log`を仕込んだ先程のFunctionsを実行したあとログを確認すると、両方ともログが残っていた。

- Stacdriver Logging
  - https://console.cloud.google.com/logs/viewer

![実行ログ](/2019/01/17-functions-log.png)

# イベントについて
Goで作成するCloud FunctionsもCloud Functionsがサポートするイベントを受け取ることができる。  
[aws-lambda-goは公式から構造体が提供](https://github.com/aws/aws-lambda-go/tree/master/events)されているが、2019/01/17時点でGCPから各イベント情報を格納する構造体はパッケージとして公開されていなそうだ。
受け取れるイベント情報の構造は以下のガイドで確認できる。

- Calling Cloud Functions
  - https://cloud.google.com/functions/docs/calling/

2019/01/17現在日本語のガイドには記載がないが、英語のガイドを確認するとGoのサンプルコードがあるので、そこから構造体定義を確認すれば良い。


下記は[ガイドに記載されている](https://cloud.google.com/functions/docs/calling/cloud-firestore)Cloud Firestoreのイベント用の構造体。

```go
// FirestoreEvent is the payload of a Firestore event.
type FirestoreEvent struct {
        OldValue   FirestoreValue `json:"oldValue"`
        Value      FirestoreValue `json:"value"`
        UpdateMask struct {
                FieldPaths []string `json:"fieldPaths"`
        } `json:"updateMask"`
}
```
![公式ページのキャプチャ](/2019/01/17-sample-code.png)


`Background Functions`実行時に受け取る`context`の中からメタデータを取得する`cloud.google.com/go/functions/metadata`パッケージは提供されている。

- GoDoc cloud.google.com/go/functions/metadata
  - https://godoc.org/cloud.google.com/go/functions/metadata

```go
import "cloud.google.com/go/functions/metadata"

// ...

meta, err := metadata.FromContext(ctx)
if err != nil {
   return fmt.Errorf("metadata.FromContext: %v", err)
}
log.Printf("Event ID: %v\n", meta.EventID)
// ...
```

# その他
go1.11ベースで動くので当然go modulesもサポートされている。また、ドキュメントでは`init()`関数や`sync.Once`を利用した実行時の初期化についても触れられていた。

- Specifying dependencies
  - https://cloud.google.com/functions/docs/concepts/go-runtime?hl=en#specifying_dependencies
- One-time initialization
  - https://cloud.google.com/functions/docs/concepts/go-runtime?hl=en#one-time_initialization



# 終わりに
軽くさわってみたが、`gcloud`コマンドの認証さえちゃんとできていれば直ぐ試せた。
とくにAPIゲートウェイなどの公開設定をしなくてもすぐHTTPリクエストの処理ができるのもよい。イベント処理を行う構造体のパッケージも提供されると無用なコピペ定義しなくても済みそうだ。

なお、Cloud Functionsは呼び出し200万回まで無料（正確には別に実行時間などの条件もある）なので誰でも試すことが出来る。Goの関数一つで試せるのでぜひ触ってみるといいと思う。

- Pricing | Documentation | Cloud Functions
  - https://cloud.google.com/functions/pricing?hl=ja


# 参考
- The Go Runtime | Documentation | Cloud Functions
  - https://cloud.google.com/functions/docs/concepts/go-runtime?hl=en
- Get Go-ing with Cloud Functions: Go 1.11 is now a supported language | Google Cloud Blog
  - https://cloud.google.com/blog/products/application-development/cloud-functions-go-1-11-is-now-a-supported-language
- GoDoc cloud.google.com/go/functions/metadata
  - https://godoc.org/cloud.google.com/go/functions/metadata
