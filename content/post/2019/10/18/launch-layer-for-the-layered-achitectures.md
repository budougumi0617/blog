+++
title= "[Go] レイヤードアーキテクチャで間違ったimportしているときに警告するlinterを作った"
date= 2019-10-18T14:36:56+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go", "OSS"]
tags = ["oss", "golang", "design", "linter", "analysis"]
keywords = ["Clean Architecture", "Onion Architecture", "Go", "Golang", "Analysis", "linter"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goでクリーンアーキテクチャ等のレイヤードアーキテクチャを実装するための静的解析ツールを作った。  
「webhandler層からusecase層を使わずに直接domain層を使わないで」というような、やってほしくない`import`をエラーにできる。

- https://github.com/budougumi0617/layer


<!--more-->

# TL;DR
- クリーンアーキテクチャなどのレイヤードアーキテクチャでは、利用できるパッケージに制限がある
  - 同じ層、あるいは1つ下の層のパッケージしか利用してはいけない
  - https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- Goは循環`import`ができないので、自然に単方向依存は満たしやすい
- しかし、層を飛び越して、2つ下の層のパッケージを直接使うような実装は言語仕様で防げない
- 今回作った静的解析ではそのような誤った`import`を静的解析で警告する
- パッケージ間の層の関係は実行時にJSON配列で渡すことで利用者独自の階層構造を読み込めるようにした

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/layer" data-iframely-url="//cdn.iframe.ly/5BGoxex"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# なぜ作ったのか
クリーンアーキテクチャやオニオンアーキテクチャなど、なんらかのレイヤードアーキテクチャの類を使って実装している人は多いのではないだろうか。
私もその中のひとりだ。

- The Clean Architecture
  - https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- The Onion Architecture : part 1
  - https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/

このようなレイヤードアーキテクチャでは、利用できるパッケージに制限がある。まず大前提として単方向の依存関係があるだろう。
また、大体の場合、各層は階層構造を持っており、それぞれのパッケージ（ライブラリ）は同じ層、あるいは1つ下の層のパッケージしか利用してはいけない。

Goは循環`import`が言語仕様でできない。そのため、Goでこのようなレイヤードアーキテクチャを実装するとき単方向依存は自然に満たしやすい。
しかし、層を飛び越して、2つ下の層のパッケージを直接使うような実装は言語仕様で防げない。今回作った静的解析ではそのような誤った`import`を静的解析で警告する。


パッケージ間の層の関係は実行時にJSON配列で渡すことで利用者独自の階層構造を読み込めるようにした。
例えば、クリーンアーキテクチャの記事に記載されている階層構造は以下のようになる。
単純に同じ層のパッケージ名を羅列し、内包する層がある場合は内部にJSON配列をまた定義すればよい。

```json
[
  "externalinterfaces",
  "web",
  "devices",
  "db",
  "ui",
  [
    "controllers",
    "gateways",
    "presenters",
    [
      "usecases",
      [
        "entity"
      ]
    ]
  ]
]
```

（`"`のエスケープがめんどくさいが、）Go1.12以上の実行環境ならば、このJSON配列を`-jsonlayer`フラグで渡すことで実行できる。
例えば以下の実行例は、`web`パッケージから`controllers`などの一つ下の層を使わずに直接2つ下の`usecases`パッケージを参照していたときに出力されるエラーだ。

```bash
$ go vet -vettool=$(which layer) -layer.jsonlayer "[\"externalinterfaces\", \
  \"web\", \"devices\", \"db\", \"ui\", [ \"controllers\", \"gateways\", \"presenters\", \
  [ \"usecases\", [ \"entity\" ] ] ] ]" ./...
# github.com/budougumi0617/clean_architecture/web
web/delete_handler.go:7:2: github.com/budougumi0617/clean_architecture/web must not include "github.com/budougumi0617/clean_architecture/usecases"

```

`repository/users`、`repository/companies`、`usecase/users`、`usecase/companies`のようなネストしたパッケージ構成を使っていた場合もチェックするように実装してある。


# 実装的な部分
実用を考えて`Analyzer`を実装したのは初めてだったが、以下のところで時間がかかった。

## JSON配列
階層構造を定義するための`-jsonlayer`フラグの引数は、YAMLで表現するとコマンドライン引数として定義を渡しにくいと思い、JSONにしてある。
キーもいらないと思い、JSON配列にしたが、JSON配列を`unmarshal`すると`[]interface{}`配列になる。
そのため、独自の`UnmarshalJSON`を実装して構造体に詰めている。

```go
// UnmarshalJSON unmarshals JSON data by custom logic.
func (l *Layer) UnmarshalJSON(data []byte) error {
	var raw []interface{}
	err := json.Unmarshal(data, &raw)
	if err != nil {
		return err
	}
	fillLayer(raw, l)

	return nil
}

func fillLayer(raw []interface{}, l *Layer) {
	l.Raw = raw
	for _, e := range raw {
		switch e := e.(type) {
		case string:
			l.Packages = append(l.Packages, e)
		case []interface{}:
			i := &Layer{}
			fillLayer(e, i)
			l.Inside = i
		}
	}
}

```


## -vettoolオプション対応
`unitchecker`パッケージを使うことで、`go vet`コマンドの`-vettool`オプション経由で利用する実行バイナリになる。

- https://godoc.org/golang.org/x/tools/go/analysis/unitchecker

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">今どきのイケてるGoの静的解析ツールを作りたかったらgo vet -vettoolオプション経由で動かすようにしたほうがいいのかな？</p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1184930409093591040?ref_src=twsrc%5Etfw">October 17, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>`

`go vet`コマンドが`-vettool`オプションを利用できるのはGo1.12以降のため、`main`パッケージの実装を`build constraints`で2種類用意しておく必要がある。
このあたりは、`gostaticanalysis/sqlrows`と同じ構成にさせてもらった。READMEの内容もほぼ踏襲している。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/gostaticanalysis/sqlrows" data-iframely-url="//cdn.iframe.ly/QxPEqNG"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## GitHub Actions
いつもはCircleCIなのだが、旬（？）なのでGitHub Actionsを使ってCIを組んでみた。慣れるまで時間がかかりそう。

- https://github.com/budougumi0617/layer/actions

## フラグの渡し方
`-vettool`オプション用のツールにしたとき、どうやってフラグの情報を渡していいのか迷った。
順を追って実行すればわかるのだが`${Analyzerの名前}.${flagの名前}`で該当のAnalyzerに値を渡すことができるらしい。

```bash
$ go vet -vettool=$(which layer) -h
usage: go vet [-n] [-x] [-vettool prog] [build flags] [vet flags] [packages]
Run 'go help vet' for details.
Run '/Users/budougumi0617/go/bin/layer -help' for the vet tool's flags.

$ layer -help
layer is a tool for static analysis of Go programs.

Usage of layer:
        layer unit.cfg  # execute analysis specified by config file
        layer help      # general help
        layer help name # help on specific analyzer and its flags

$ layer help layer
layer: layer checks whether there are dependencies that illegal cross-border the layer structure. The layer structure is defined as a JSON array using the -jsonlayer option.

Analyzer flags:

  -layer.jsonlayer string
        jsonlayer defines layer hierarchy by JSON array (default "[ \"external\",\"db\",\"ui\", [ \"controllers\", [ \"usecases\", [ \"entity\" ] ] ] ]")

```

## テストコードは無視する
さすがにテストコードまで厳密にレイヤードアーキテクチャする必要はないと思っている。
なので、`_test.go`でファイル名が終わる場合は無視する。
実行時唯一の情報である[`analysis.Pass`](https://godoc.org/golang.org/x/tools/go/analysis#Pass)構造体からファイル名を取得する方法は[@podhmo](https://twitter.com/podhmo)さんの記事を参考にした。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fpod.hatenablog.com%2Fentry%2F2018%2F03%2F05%2F070458" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

# 終わりに
前々から作りたいなと思っていて、時間ができたので作ってみた。
脳内設計は前からしていたものの、実装自体はだいたい8時間以内で完成していると思う。
一度慣れれば問題ない部分も多いので、次の作るときはもう少しは早く終わりそうだ。
静的解析ツール作成はインフラの準備もいらないし、言語仕様（構造）の勉強にもなるし面白い。

# 参考
- https://github.com/budougumi0617/layer
- [The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [The Onion Architecture : part 1](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/)
- [unitchecker | GoDoc](https://godoc.org/golang.org/x/tools/go/analysis/unitchecker)
- https://github.com/gostaticanalysis/sqlrows
- [goのast.Fileからファイル名を取得する方法](https://pod.hatenablog.com/entry/2018/03/05/070458)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07FSBHS2V&linkId=c46b6daefa3b3e617c886f4bea107fc7"></iframe>
