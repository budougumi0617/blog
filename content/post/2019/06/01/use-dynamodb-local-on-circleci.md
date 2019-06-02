+++
title= "DynamoDBを操作するコードをamazon/dynamodb-localコンテナとCircleCIを使ってCIする #aws"
date= 2019-06-02T18:07:58+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["aws", "circleci"]
tags = ["aws", "circleci", "ci", "test", "dynamodb"]
keywords = ["DynamoDB", "Go", "CircleCI", "AWS"]
twitterImage = "logos/aws.png"
+++

DynamoDBを操作するコードをCircleCI上でテストする方法をまとめた。  
dynamodb-localをCircleCI上で起動することで、実際にDBアクセスする状態でテストが実行できる。

<!--more-->

# TL;DR
- AWSはローカル環境で動作確認できるようにDynamoDBを起動できるコンテナを配布している
  - https://hub.docker.com/r/amazon/dynamodb-local/
- このコンテナを使ってオフラインでDybamoDBを使ったテストを実行できる
- CircleCI上でもこのコンテナを起動して、DynamoDBを使ったCIを定義した
  - ダミーで良いので`AWS_ACCESS_KEY_ID`と`AWS_SECRET_ACCESS_KEY`を設定しておく
  - 念のためテスト実行前に`nc`コマンドでDynamoDBと疎通できるまで待機するようにした

なお、サンプルコードは以下になる。
GoのLambdaからDynamoDBを操作することを想定している。

- budougumi0617/lambda-go-dynamodb-local
  - https://github.com/budougumi0617/lambda-go-dynamodb-local/tree/add-circleci

CircleCIの実行結果はこちら。

- https://circleci.com/gh/budougumi0617/lambda-go-dynamodb-local/7

# はじめに
DynamoDBはAWSが提供するNoSQL DBだ。Lambdaなどと組み合わせて使われることも多い。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://aws.amazon.com/jp/dynamodb/" data-iframely-url="//cdn.iframe.ly/O3qqkDe?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

LambdaとDyanamoDBを使ったコードを書こうと思ったのだが、できればローカルで動作確認が完了すると嬉しい。  
DynamoDBクローンのDockerイメージがあるとのことだったので、こちらを利用することにした。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://aws.amazon.com/jp/about-aws/whats-new/2018/08/use-amazon-dynamodb-local-more-easily-with-the-new-docker-image/" data-iframely-url="//cdn.iframe.ly/R0M5dLT?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

開発するにあたり、CircleCI上のテストでもこのDockerイメージを使うことにした。

# CircleCIで実行するテストコード
まず、CircleCIで実行するリポジトリは以下になる。

- budougumi0617/lambda-go-dynamodb-local
  - https://github.com/budougumi0617/lambda-go-dynamodb-local/tree/add-circleci

Goで書かれたテストコードはこちら。テスト対象のコードを実行する前にDynamo DB上にテーブルを作成、作成したテーブルにアイテムを保存しておく。  
また、呼び出しているテスト対象のコード自体はDynamoDB上からアイテムを取得するコードだ。

```go
// コード全体はこちら
// https://github.com/budougumi0617/lambda-go-dynamodb-local/blob/add-circleci/user/main_test.go

func TestGetUser(t *testing.T) {
	ep := os.Getenv("DYNAMODB_ENDPOINT")
	r := os.Getenv("AWS_REGION")
	tn := os.Getenv("DYNAMODB_TABLE_NAME")

	dyn := dynamodb.New(
		session.Must(
			session.NewSession(
				&aws.Config{
					Endpoint: aws.String(ep),
					Region:   aws.String(r),
				},
			),
		),
	)

	if err := deleteTestData(dyn, tn); err != nil {
		t.Logf("delete table failed %v\n", err)
	}

	if err := createTestData(dyn, tn); err != nil {
		t.Fatalf("table created failed rr: %v\n", err)
	}

	h := generateHandler(ep, r, tn)
	in := events.APIGatewayProxyRequest{}
	in.PathParameters = make(map[string]string)
	in.PathParameters["id"] = "1"

	res, err := h(in)
	if err != nil {
		t.Fatalf("err: %v\n", err)
	}
	t.Logf("result = %+v\n", res)
}

func createTestData(db *dynamodb.DynamoDB, tn string) error {
	input := &dynamodb.CreateTableInput{
		// ...
	}

	if _, err := db.CreateTable(input); err != nil {
		return err
	}

	// PutItem
	putParams := &dynamodb.PutItemInput{
		// ...
	}

	if _, err := db.PutItem(putParams); err != nil {
		return err
	}
	return nil
}

func deleteTestData(db *dynamodb.DynamoDB, tn string) error {
	input := &dynamodb.DeleteTableInput{
		TableName: aws.String(tn),
	}
	_, err := db.DeleteTable(input)
	if err != nil {
		return err
	}
	return nil
}
```

このコードをモックを使わずCircleCI上で実行する。

## ローカルに起動したDynamoDBを使ってテストを実行する

ちなみに、`amazon/dynamodb-local`コンテナを使えばローカルにDynamoDBを起動することもできる。
ローカルのDynamoDBを使って上記のテストコードを実行するときは以下のようになる。


```
$ cat docker-compose.yaml
version: '3'

services:
  dynamodb:
    image: amazon/dynamodb-local
    container_name: dynamodb
    ports:
      - 18000:8000
$ docker-compose up -d
$ DYNAMODB_ENDPOINT="http://localhost:18000" go test ./...

```

# CircleCIの設定
CircleCIの設定は以下のようになる。関連する設定部分はこのを個別に解説を入れる。

```yaml
# https://github.com/budougumi0617/lambda-go-dynamodb-local/blob/add-circleci/.circleci/config.yml

version: 2.1
executors:
  default:
    docker:
      # CircleCI Go images available at: https://hub.docker.com/r/circleci/golang/
      - image: circleci/golang:1.12.5
      - image: amazon/dynamodb-local

    working_directory: /go/src/github.com/budougumi0617/lambda-go-dynamodb-local

    # Environment values for all container
    environment:
      - GO111MODULE: "on"
      - TEST_RESULTS: /tmp/test-results # path to where test results will be saved
      - AWS_REGION: "test-region"
      - DYNAMODB_ENDPOINT: "http://localhost:8000"
      - DYNAMODB_TABLE_NAME: "User"

jobs:
  test:
    executor:
      name: default
    steps:
      - checkout
      - run: mkdir -p $TEST_RESULTS
      - restore_cache:
          name: Restore go modules cache
          keys:
              - v1-mod-{{ .Branch }}-{{ checksum "go.mod" }}
      - run:
          name: Vendoring
          command: go mod download
      # カバレッジの集計などをするためのツールのインストール
      - run:
          name: Install test report tool
          command: go get github.com/jstemmer/go-junit-report
      - save_cache:
          name: Save go modules cache
          key: v1-mod-{{ .Branch }}-{{ checksum "go.mod" }}
          paths:
              - /go/pkg/mod/cache
      - run:
          name: Wait for DynamoDB
          command: |
            for i in `seq 1 10`;
            do
              nc -z localhost 8000 && echo Success && exit 0
              echo -n .
              sleep 1
            done
            echo Failed waiting for DyanmoDB Local && exit 1
      - run:
          name: Run all unit tests
          command: |
            trap "go-junit-report <${TEST_RESULTS}/go-test.out > ${TEST_RESULTS}/go-test-report.xml" EXIT
            go test ./... | tee ${TEST_RESULTS}/go-test.out
      - store_artifacts:
          path: /tmp/test-results
          destination: raw-test-output
      - store_test_results:
          path: /tmp/test-results

workflows:
  build-and-test:
    jobs:
      - test
```

## CircleCIでamazon/dynamodb-localイメージを起動する
CircleCIは一連のジョブを実行するイメージの他にDBなどのサブイメージを起動できる。  
メインイメージからサブイメージへのアクセスは`localhost`経由で可能だ。

```yaml
executors:
  default:
    docker:
      # CircleCI Go images available at: https://hub.docker.com/r/circleci/golang/
      - image: circleci/golang:1.12.5
      - image: amazon/dynamodb-local
    # Environment values for all container
    environment:
      - DYNAMODB_ENDPOINT: "http://localhost:8000"
```
## AWS_ACCESS_KEY_IDとAWS_SECRET_ACCESS_KEYを設定する
CircleCIのリポジトリの設定にある`Environment Variables`で`AWS_ACCESS_KEY_ID`と`AWS_SECRET_ACCESS_KEY`を設定しておく。  
`dynamodb-local`は起動時と接続時に認証情報を内部で利用している。実際にAWSにアクセスできる有効な文字列でなくていいので、認証情報を設定しておく必要がある。



## ncコマンドで疎通を確認する
MySQLなどは`dockerize`コマンドを使うのだが、`dockerize`コマンドで`http`プロトコルの確認をするときは`200`ステータスが必要になる。  
`dynamodb-local`への疎通確認を`http`プロトコルで行なうと認証をつけないと`400`エラーになるため、今回は利用をやめた。  
そのため、今回は単純にポートがリッスンしていることを確認する`nc`コマンドを使った。

```yaml
      - run:
          name: Wait for DynamoDB
          command: |
            for i in `seq 1 10`;
            do
              nc -z localhost 8000 && echo Success && exit 0
              echo -n .
              sleep 1
            done
            echo Failed waiting for DyanmoDB Local && exit 1
```

以上の設定を行えば、CircleCI上でDynamoDBを利用したテストを実行できる。

- https://circleci.com/gh/budougumi0617/lambda-go-dynamodb-local/7

# 終わりに
今回は`amazon/dynamodb-local`コンテナを使ってCircleCIでDynamoDBを操作するテストを実行した。  
`dynamodb-local.jar`をCircleCI上で起動して同様の操作をする記事はあったのだが、このためだけにJavaの環境を用意するのは他の言語を使って開発しているときは少しめんどくさい。  
今回の記事はCircleCIの設定を調べるよりローカルでうまくDynamoDBと疎通するコードを作るほうが難しかった（正確にいうと、`sam local start-api`コマンドで実行したLambdaから`dynamodb-local`で起動したDynamoDBにアクセスするのに苦戦していた）。  
LambdaとDynamoDBの組み合わせは料金もほぼかからない組み合わせのため、もう少し勉強しておくつもりだ。

# 参考
- [budougumi0617/lambda-go-dynamodb-local](https://github.com/budougumi0617/lambda-go-dynamodb-local/tree/add-circleci)
- [amazon/dynamodb-local | DockerHub](https://hub.docker.com/r/amazon/dynamodb-local/)
- [dynamodb | AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/dynamodb/index.html#cli-aws-dynamodb)
- [AWS SAM + DynamoDB Local + Go で始めるサーバレスアプリケーション開発](https://www.m3tech.blog/entry/advent-calendar-2018-6)
- [CircleCI上でdynamodb-localを使ったgo testを実行する](https://kyokomi.hatenablog.com/entry/2016/09/09/145906)
- [AWS-SAM-Localを使った時にハマったメモ](http://blog.deresuke.org/2018-01-05/)
