+++
title = "go listで依存パッケージを一覧する。-tagsで依存パッケージを切り替える #go"
date = 2018-09-21T09:18:17+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "build","gotips"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


今更だがGoでビルドタグで依存パッケージをどう制御できるのか確認した。  
また、依存パッケージの一覧を確認するのには`go list`コマンドの使い方を調べた。

<!--more-->

# TL;DR
- `go list -f '{{join .Deps "\n"}}'`コマンドで依存パッケージの情報を取得できる
- `go`コマンドは`go build`/`go test`コマンド以外のときもビルドタグに基づいた結果を出力する
- ビルドタグによってビルドに含まれないコードの依存パッケージはビルドにも含まれない

確認で利用したソースコードは以下。

- [https://github.com/budougumi0617/sandbox-go-cloud@679665c](https://github.com/budougumi0617/sandbox-go-cloud/tree/679665ce3573ad2f83642e2613a95424c1647456)


# ソースコードの事前準備
確認では以下の構成のソースコードを作成した。  
簡易の構成だが、AWS関係のパッケージに依存した`aws`パッケージと、GCP関係のパッケージに依存した`gcp`パッケージがある。

```bash
$ tree
.
├── LICENSE
├── README.md
├── aws           # !gcpタグ付きのコード
│   └── setup.go  # AWS SDKに依存しているコード
├── aws.go
├── gcp
│   └── setup.go  # GCP SDKに依存しているコード
├── gcp.go        # gcpタグ付きのコード
├── main.go
└── sandbox-go-cloud
```

`main.go`では`test()`関数を呼ぶ。


```go
package main

func main() {
	// tags=gcpのときはgcp.goのtest()が呼ばれる
	// tagsがなければaws.goのtest()が呼ばれる
	test()
}
```

`test()`関数は`./aws.go`と`./gcp.go`に定義した。  
２つのファイルはビルドタグによって必ず片方しかビルドに含まれないので、同じ関数があっても矛盾しない。

## サブパッケージをimportするファイルにビルドタグをつける
- https://golang.org/pkg/go/build/#hdr-Build_Constraints

Goは他言語の`#ifdef`のように、ビルドタグを使ってファイル単位でビルドスイッチを定義することができる。  
利用するときはファイルの先頭に`// +build`から始まるビルドタグを書く。必ず二行目は空行にする必要がある。複数のビルドタグを付けることも可能。

`gcp.go`は`gcp`タグがついている時にしかビルドに含まれないファイルになる。


```go
// +build gcp

package main

import (
	// go-cloud経由でGCPのSDKに依存しているサブパッケージ
	"github.com/budougumi0617/sandbox-go-cloud/gcp"
)

func test() {
	gcp.Print()
}
```

ビルドタグの定義には倫理式も使えるので、`aws.go`は`gcp`タグがついて「いない」時にビルドに含まれるファイルになる。

```go
// +build !gcp

package main

import (
	// AWSのSDKに依存しているサブパッケージ
	"github.com/budougumi0617/sandbox-go-cloud/aws"
)

func test() {
	aws.Print()
}
```

ビルドタグの定義により、`aws.go`と`gcp.go`はビルド時にいずれしか含まれない。それぞれ別のサブパッケージに依存している。  
また、今回は結果をわかりやすくするため、GCP関係の依存パッケージをローカルから取り除いている。（GCP関係の依存があるとパッケージが見つからずにビルドが失敗する）

# go listで依存パッケージの一覧を取得する
- https://golang.org/cmd/go/#hdr-List_packages_or_modules

`go list`コマンドは`text/template`のようなコマンドオプションを使うと、出力情報を制御できる。  
また、`go`コマンドは`go build`や`go test`コマンド以外のときもビルドタグに基づいた結果を出力する。

まず、`gcp`タグ無しで依存パッケージの一覧を出力する。  
`go list`コマンドでは[こちら](https://qiita.com/suin/items/edf59157ba010c04b50e)の記事を参考に標準パッケージ以外の依存パッケージを出力するようにしている。  
結果はたしかにAWS関係の依存のみ出力された。

```bash
$ go list -f '{{join .Deps "\n"}}' | xargs go list -f '{{if not .Standard}}{{.ImportPath}}{{end}}'
github.com/aws/aws-sdk-go/aws
github.com/aws/aws-sdk-go/aws/awserr
...
github.com/google/go-cloud/blob/s3blob
```

`gcp`タグをつけて`go list`コマンドを実行する。  
(GCP関連のパッケージは`go get`していないのでエラーが出ているが、）GCP関連のパッケージへの依存を確認できた。  
また、AWS関係の依存はなくなっていた。

```bash
$ go list -tags=gcp -f '{{join .Deps "\n"}}' | xargs go list -f '{{if not .Standard}}{{.ImportPath}}{{end}}'
can't load package: package cloud.google.com/go/storage: cannot find package "cloud.google.com/go/storage" in any of:
	/usr/local/opt/go/libexec/src/cloud.google.com/go/storage (from $GOROOT)
	/Users/shimizu-yoichiro/go/src/cloud.google.com/go/storage (from $GOPATH)
...
github.com/google/go-cloud/gcp
github.com/google/go-cloud/wire
```

# go buildして依存を確認する

今度は実際にビルドしてみる。ビルドタグがないときのビルドはAWS関係のパッケージにしか依存関係がないはずなので成功する。

```bash
$ go build
```

ビルドタグを付与してビルドすると、GCP関係のパッケージが見つからずビルドエラーになる。  
ビルドタグで依存パッケージを制御することができた。

```bash
$ go build -tags=gcp
../../google/go-cloud/blob/gcsblob/gcsblob.go:31:2: cannot find package "cloud.google.com/go/storage" in any of:
	/usr/local/opt/go/libexec/src/cloud.google.com/go/storage (from $GOROOT)
	/Users/shimizu-yoichiro/go/src/cloud.google.com/go/storage (from $GOPATH)
../../google/go-cloud/gcp/gcp.go:24:2: cannot find package "golang.org/x/oauth2" in any of:
	/usr/local/opt/go/libexec/src/golang.org/x/oauth2 (from $GOROOT)
...
```

# 終わりに
今回はビルドタグを使って、依存パッケージを切り替えることを確認した。  
なぜこんなことをしているかたというと、go-cloudやDependency Injectionの検証をしていたから。この辺は別の場所で公開する。

- https://godoc.org/github.com/google/go-cloud

また、`go list`はオプションをつけて実行しないと大した情報が出てこないので、使うときは`-f`オプションをちゃんと使ったほうがよさそう。



# 参考
- Package build | Build Constraints
  - https://golang.org/pkg/go/build/#hdr-Build_Constraints
- Command Go | List packages or modules
  - https://golang.org/cmd/go/#hdr-List_packages_or_modules
- Go言語: 依存するパッケージ一覧を調べたい
  - https://qiita.com/suin/items/edf59157ba010c04b50e
