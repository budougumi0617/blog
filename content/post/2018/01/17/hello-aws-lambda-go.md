+++
title= "AWS Lambda Go早めぐり(LambdaContext, Logging, Error...)"
date= 2018-01-17T07:33:10+09:00
draft = false
slug = ""
categories = ["Go", "Lambda", "AWS"]
tags = ["golang", "aws", "lambda", "serverless"]
author = "budougumi0617"
+++

AWS LambdaでGoが正式にサポートされた！！

**AWS Lambda Supports Go**  
https://aws.amazon.com/jp/about-aws/whats-new/2018/01/aws-lambda-supports-go/

**aws/aws-lambda-go**  
https://github.com/aws/aws-lambda-go

2018/01/17現在、日本語版はまだだが、英語のDocsは公開されているので、それを基にGoでLambdaを動かす。

**Programming Model for Authoring Lambda Functions in Go**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/go-programming-model.html

**Announcing Go Support for AWS Lambda**  
https://aws.amazon.com/jp/blogs/compute/announcing-go-support-for-aws-lambda/


# TL;DR
- 基本的にコードファイルひとつで実行バイナリまで作れる
- ハンドラ定義は5パターンのみ（`TIn`, `TOut`は`encoding/json`が使える構造体）
- 渡される`Context`の中には`LambdaContext`が入っている
- ログは`fmt`や`log`を使っていればよしなにしてくれる
- エラー処理は`error`や`panic`など標準のものが使える
- `os.Getenv`で環境変数の確認ができる

実装したコードは以下  
https://github.com/budougumi0617/sandbox-aws-lambda-go/blob/master/hello-lambda/main.go


```go
package main

import (
	"context"
	"errors"
	"fmt"
	"log"
	"os"

	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
	"github.com/aws/aws-lambda-go/lambdacontext"
)

func HelloLambdaHandler(ctx context.Context, config events.ConfigEvent) (string, error) {
	lc, ok := lambdacontext.FromContext(ctx)
	if !ok {
		return "", errors.New("Cannot find LambdaContext")
	}

	log.Printf("log:InvokedFunctionArn = %s\n", lc.InvokedFunctionArn)
	fmt.Printf("fmt:InvokedFunctionArn = %s\n", lc.InvokedFunctionArn)
	log.Printf("config version is %v\n", config.Version)
	log.Printf("\"%s\" executes on \"%s\".\n", os.Getenv("LAMBDA_TASK_ROOT"), os.Getenv("AWS_EXECUTION_ENV"))

	return "function finished", nil
}

func main() {
	lambda.Start(HelloLambdaHandler)
}
```

# 前提・準備
とりあえず動かすだけなのでzipでLambdaにコードをアップしてテストイベントで実行してみる。
事前に空のLambda関数を作っておく。自分の設定は以下。

- コードエントリタイプ
  - .ZIPファイルをアップロード
- ランタイム
  - Go 1.x
- ハンドラ
  - main
- 実行ロール
  - [IAM](https://console.aws.amazon.com/iam/home?region=ap-northeast-1#/roles)で空のロールを作っておきそれを使う
- テストイベント
  - イベントテンプレートの`AWS Config Change Triggered Rule`の値を少し変えたものを用意した。

```json
{
  "configRuleId": "config-rule-0123456",
  "version": "6.17",
  "configRuleName": ...
}
```

もろもろのGoの開発環境は済んでいると想定する。


# ハンドラ定義は5パターンのみ（`TIn`, `TOut`は`encoding/json`が使える構造体）
**Lambda Function Handler (Go)**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/go-programming-model-handler-types.html

ハンドラには`context.Context`やイベント情報が渡される。定義出来るパターンは以下。

- `func ()`
- `func () error`
- `func (TIn), error`
- `func () (TOut, error)`
- `func (context.Context) error`
- `func (context.Context, TIn) error`
- `func (context.Context) (TOut, error)`
- `func (context.Context, TIn) (TOut, error)`

引数は0〜2個まで。引数を指定するときは`context.Context`を第一引数にするのが必須。  
戻り値も0〜2個。最終戻り値は`error`になる形。  
`TIn`と`TOut`は`encoding/json`で`Unmarshal`できる必要がある。  
AWS内のイベントは既に構造体が定義されているので、それをつかえる。  

https://godoc.org/github.com/aws/aws-lambda-go/events

今回は`events.ConfigEvent`を使ってみた。

```go
package events

// ConfigEvent contains data from an event sent from AWS Config
type ConfigEvent struct {
  ...
  Version          string `json:"version"`
}
```


# 渡される`Context`の中には`LambdaContext`が入っている
**The Context Object (Go)**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/go-programming-model-context.html

https://godoc.org/github.com/aws/aws-lambda-go/lambdacontext

ハンドラに渡される`context.Context`の中には、`LambdaContext`が入っていて、AWSの情報がよしなに入っている。

```go
	lc, ok := lambdacontext.FromContext(ctx)
	if !ok {
		return "", errors.New("Cannot find LambdaContext")
	}
	fmt.Printf("InvokedFunctionArn = %s\n", lc.InvokedFunctionArn) // InvokedFunctionArn = arn:aws:lambda:ap-no...

```

[lambdacontext.NewContext](https://godoc.org/github.com/aws/aws-lambda-go/lambdacontext#NewContext)を使って
自分で`Context`の中に`LambdaContext`を入れることも出来る(ハンドラ内でgoroutine回したりする時用？)

# ログは`fmt`や`log`を使っていればよしなにしてくれる
**Logging (Go）**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/go-programming-model-logging.html

ログは他の言語で実装したとき同様標準出力に出力すれば`CloudWatch`に保存される。`log`を使えばタイムスタンプを付与しておく事もできる。

```go
log.Printf("log:InvokedFunctionArn = %s\n", lc.InvokedFunctionArn) // 2018/01/16 23:07:46 log:Invoked...
fmt.Printf("fmt:InvokedFunctionArn = %s\n", lc.InvokedFunctionArn) // fmt:Invoked...

```

# エラー処理は`erro`や`panic`など標準のものが使える
**Function Errors (Go)**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/go-programming-model-errors.html

エラー処理は普通のGoとあまり変わらないようだ。ちゃんと試せてはいない。
`panic`を仕込むとStackTraceなども取れるらしい？

以下、上記公式の内容を引用。

```json
{
    "errorMessage": "Something went wrong",
       "errorType": "errorString",
    "stackTrace": [
            {
            "path": "github.com/aws/aws-lambda-go/lambda/function.go",
            "line": 27,
            "label": "(*Function).Invoke.function"
        },
 ...

    ]
}
```

# `os.Getenv`で環境変数の確認ができる
**Using Environment Variables (Go)**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/go-programming-model-env-variables.html

他の言語と同様、`os.Getenv`で環境変数を取得できる。

```go
fmt.Printf("\"%s\" executes on \"%s\".\n", os.Getenv("LAMBDA_TASK_ROOT"), os.Getenv("AWS_EXECUTION_ENV"))  // "/var/task" executes on "".
```

ちなみに`AWS_EXECUTION_ENV`がどうなるか書いてなかったので、興味本位で見ようと思ったら本当に設定されてなかった（？）

**Environment Variables Available to Lambda Functions**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/current-supported-versions.html#lambda-environment-variables


2018/01/17現在、用意されている内容はこれくらい。

# デプロイ・動作確認
上記内容を全て実装したのが、冒頭に記載した以下のリンク先のコード  
https://github.com/budougumi0617/sandbox-aws-lambda-go/blob/master/hello-lambda/main.go

これをAWS Lambda上で動かしてみる。Goの(一番簡単な方法をする)場合はビルドしてzipに入れてアップするだけ。
Linux用にビルドするのと、「前提・準備」で設定した「ハンドラ」と同じ名前のバイナリをするところだけ注意する。
これをLambdaのWebコンソールからアップロードすれば実行できる

```
$ GOOS=linux go build -o main
$ zip deployment.zip main
```

作成したテストイベントで実行した結果がこちら。
```
START RequestId: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXX Version: $LATEST
2018/01/16 23:07:46 log:InvokedFunctionArn = arn:aws:lambda:ap-northeast-1:XXXXXXXXXXXXXX:function:myFirstLambda
fmt:InvokedFunctionArn = arn:aws:lambda:ap-northeast-1:XXXXXXXXXXXX:function:myFirstLambda
2018/01/16 23:14:26 config version is 6.17
2018/01/16 23:14:26 "/var/task" executes on "".
END RequestId: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXX
REPORT RequestId: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXX	Duration: 0.59 ms	Billed Duration: 100 ms 	Memory Size: 128 MB	Max Memory Used: 27 MB
```

![Lambdaの実行結果](/2018/01/result-my_first_lambda.png)

動いた！`LambdaContext`の中身も確認できている。


# 終わりに
この発表を聞いて初めてAWSの無料枠に入ったレベルだったので、今回はひとつのLambda内に閉じた実装だけ確かめた。
Goでの実装はコードひとつだけでビルドできるので、非常に簡単だった。
(正直コーディングの時間より[初回登録](https://aws.amazon.com/jp/register-flow/)時の電話認証のほうが時間かかっている。)  
自分はあまりクラウドに詳しくないのだが、これくらい小さく始められるのは嬉しい。CloudWatchにログ流したり、別のサービスのイベント拾ったり、少しずつ視野を広げられそう。  
設定ファイルを置けばAWS CodePipelineなどを使って自動デプロイなども出来るらしいので、後日確かめてみたい。
(冒頭の[ブログ](https://aws.amazon.com/jp/blogs/compute/announcing-go-support-for-aws-lambda/)に詳しく書かれている。)

**Creating a Deployment Package (Go）**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/lambda-go-how-to-create-deployment-package.html


# 参考
**aws/aws-lambda-go**  
https://github.com/aws/aws-lambda-go

**GoDoc:aws/aws-lambda-go**  
https://godoc.org/github.com/aws/aws-lambda-go

**AWS Lambda Supports Go**  
https://aws.amazon.com/jp/about-aws/whats-new/2018/01/aws-lambda-supports-go/

**Announcing Go Support for AWS Lambda**  
https://aws.amazon.com/jp/blogs/compute/announcing-go-support-for-aws-lambda/

**Programming Model for Authoring Lambda Functions in Go**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/go-programming-model.html

**Creating a Deployment Package (Go）**  
https://docs.aws.amazon.com/en_us/lambda/latest/dg/lambda-go-how-to-create-deployment-package.html

**AWS SAMを利用してGolangなLambdaをデプロイする**  
https://qiita.com/ikeisuke/items/3c0c422888ae8ae09831

**awsのlamdaがGo言語対応したので雑にsampleを動かしてみる**  
https://qiita.com/ieee0824/items/5bf3f9649f2378c6f2e5

**budougumi0617/sandbox-aws-lambda-go**  
https://github.com/budougumi0617/sandbox-aws-lambda-go

