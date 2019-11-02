+++
title= "Go Modulesの概要とGo1.12に含まれるModulesに関する変更 #golangjp #go112party"
date= 2019-02-15T10:43:23+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "modules"]
keywords = ["Go1.12", "Go Modules", "Go", "vendoring"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

今更ながらGo Modulesについて簡単にまとめた。
そして今月(2019年2月)にリリースされる予定のGo1.12に含まれるGo Modules関連の仕様変更について調べた。

<!--more-->

なお、この記事はGo 1.12 Release Party登壇に関する補足資料となる。


|イベント名|Go 1.12 Release Party in Tokyo w/ Fukuoka&Umeda|
|---|---|
|URL|https://gocon.connpass.com/event/118022/|
|会場|株式会社メルカリ 東京都港区六本木6-10-1(六本木ヒルズ森タワー18F)|
|日時|2019/02/15(金) 19:30 〜 22:00|
|ハッシュタグ|[#go112party](https://twitter.com/hashtag/go112party)|

  

<br><script async class="speakerdeck-embed" data-id="025e3f94d6ef4685bd8b3cca762b8e38" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>


# TL;DR
- Go Modules(vgo)はGo1.11から導入され始めたGoの新しいバージョン管理
- Go1.12ではまだ有効にはなっていない(Go1.13からはデフォルトで有効になる）
- Go Modulesの概要とTipsなどを簡単にまとめた
- Go1.12のModules関連の変更をDockerを動かして確認してみた
  - `GOPATHi`外で`go.mod`がなくても`go run`可能
  - `replace`ディレクティブで依存パッケージをローカルのコードを使って解決
  - etc...

確認に利用したDockerfileやスクリプトは以下のリポジトリにある。

- https://github.com/budougumi0617/gomodules-explore

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/gomodules-explore" data-iframely-url="//cdn.iframe.ly/XQI0aX5"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# Modulesの概要
Go Modulesは2018年2月にRuss Coxさんが発表したGoのプロジェクトの依存パッケージに対するバージョン管理構想だ。
最初は`vgo`と呼ばれ、Goとは独立したCLIツールとして提供されており、Go1.11からGo本体に試験的に導入された。

- Go & Versioning | research!rsc
  - https://research.swtch.com/vgo

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/golang/vgo" data-iframely-url="//cdn.iframe.ly/LgJYXfz"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`vgo`発表前のGoのバージョン管理は以下のような手法が取られていた。

- `go get`で`go1`タグ・ブランチもしくは最新の`master`ブランチを取得する
  - [go getで取得されるコードはmasterブランチ(HEAD)がデフォルトではない #golang](/2018/05/10/go-get-from-go1-tag-or-branch/)
- `glide`や`dep`などのサードパーティ製ツールでバージョン管理をする
  - https://github.com/Masterminds/glide
  - https://github.com/golang/dep

2017年、2018年に新規にプロジェクトを作るのならば`dep`で管理することが多かったように思う。
（[vvakameさんの記事](https://blog.vvaka.me/2018/06/08/vgo/)を見たりして私は`vgo`への移行は様子見していた。）

2019/02/16追記 Modulesはvgoよりはいい子になったとのこと。
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">vgo？go modulesはvgoに比べるといい子になったよ！って伝えておいて！</p>&mdash; わかめ@毎日猫がいる (@vvakame) <a href="https://twitter.com/vvakame/status/1096366500040994816?ref_src=twsrc%5Etfw">2019年2月15日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


## Minimal Version Selection
Go ModulesはSemantic Versioningに基づいてモジュールの管理を行なう。Semantic Versioningとは、`vX.X.X`というようなバージョン番号の定義方法だ。

- Semantic Versioning 2.0.0
  - https://semver.org/

![Semantic Versioning](/2019/02/15_semantic.png)
[Semantic Import Versioning](https://research.swtch.com/vgo-import)より引用


Go以外のプログラミング言語・フレームワークでも依存ライブラリのバージョン管理は行われているが、Go Modulesは可能な限り古いバージョンを採用するのが特徴的だ。

- Minimal Version Selection | research!rsc
  - https://research.swtch.com/vgo-mvs

## ２つのモード
- Module-aware go get
  - https://golang.org/cmd/go/#hdr-Module_aware_go_get

Go1.11、Go1.12のGo ModulesはGOPATH modeとmodule-aware modeという２つのモードを有している。
大まかな違いは以下の通り。

- GOPATH mode
  - バージョン 1.10 までと同じ挙動をする。
  - 標準pkg以外を全部 `GOPATH` 以下のディレクトリで管理する
  - パッケージの管理はリポジトリの最新リビジョンのみが対象となる
- module-aware mode
  - 標準pkg以外の全てのパッケージをモジュールとして管理する
  - モジュールの管理やビルドが任意のディレクトリで可能になる，
  - モジュールはリポジトリのバージョンタグまたはリビジョン毎に管理される

2つのモードは`GO111MODULE`という環境変数で切り替わる。

- `on`	常にmodule-aware modeで動作する
- `off`	常にGOPATH modeで動作する
- `auto` `$GOPATH` 配下ではGOPATH modeで，それ以外のディレクトリではmodule-aware modeで動作する

Go1.11のときはデフォルトの挙動が`auto`だったので、何もしなければ今までと同じ挙動をする。
Go1.12もGo1.11と同じように`auto`がデフォルト状態になっている。ただし半年後にリリースされるGo1.13では`GO111MODULE=on`がデフォルトとなる予定だ。

- Go Modules in 2019
  - https://blog.golang.org/modules2019

> Go 1.12, scheduled for February 2019, will refine module support but still leave it in auto mode by default.
> Our aim is for Go 1.13, scheduled for August 2019, to enable module mode by default (that is, to change the default from auto to on) and deprecate GOPATH mode.

# go modコマンド
Modulesの機能はモジュールのルートディレクトリに配置する`go.mod`ファイルと`go mod`コマンドから利用することができる。
すでに解説記事がいくつかあるのでここでは`go.mod`ファイルについてだけ触れる。
`go mod`コマンドの概要は以下の通り。

```bash
$ go help mod
Go mod provides access to operations on modules.

Note that support for modules is built into all the go commands,
not just 'go mod'. For example, day-to-day adding, removing, upgrading,
and downgrading of dependencies should be done using 'go get'.
See 'go help modules' for an overview of module functionality.

Usage:

	go mod <command> [arguments]

The commands are:

	download    download modules to local cache
	edit        edit go.mod from tools or scripts
	graph       print module requirement graph
	init        initialize new module in current directory
	tidy        add missing and remove unused modules
	vendor      make vendored copy of dependencies
	verify      verify dependencies have expected content
	why         explain why packages or modules are needed

Use "go help mod <command>" for more information about a command.
```

依存関係は`go.mod`ファイルに書き出される。`go.mod`ファイルは`go mod init`コマンドで作成することができる（`go mod download`をすると`Gemfile.Lock`や`package-lock.json`のように`go.sum`ファイルも生成される）。


いくつかの依存関係を含んだ`go.mod`ファイルは以下のようになる。

```
module example.com/m

replace (
	example.com/a => ./a
	example.com/a/b => ./b
)

require (
	example.com/a/b v0.0.0-00010101000000-000000000000
	example.com/v v1.12.0
	example.com/x/v3 v3.0.0-00010101000000-000000000000
	example.com/y v0.0.0-00010101000000-000000000000
)
```

`go.mod`の中のディレクティブ（構成要素）の概要は以下の通り（Go1.12では後述する`go`ディレクティブも増える）。

- `module` ルートディレクトリのモジュール名
- `require` 必要なモジュール名とバージョン名を指定する
- `exclude` 明示的に除外するモジュールを指定する
- `replace` `require`で指定したモジュール名を置き換える

`go mod`コマンドの利用法は以下のブログが日本語で詳しい。

- モジュール対応モードへの移行を検討する
  - https://text.baldanders.info/golang/go-module-aware-mode/

また、`go/src/cmd/go/testdata/script`に`go`コマンド自体のテストスクリプトがあるのでそこから`go mod`コマンドの使い方を探ることもできる。

- go/src/cmd/go/testdata/script
  - https://github.com/golang/go/tree/master/src/cmd/go/testdata/script

# Tips
Go Modulesに関するTipsは以下の通り。

- `go mod`で取得したバイナリなどは`$GOPATH/pkg/mod/`以下にキャッシュされている
  - なのでCIの高速化でビルドキャッシュをするときはこのディレクトリをキャッシュする必要がある
  - キャッシュを削除するときは`go clean -cache`コマンドで削除できる
  - `go mod vendor`コマンドで`dep`のように`vendor`ディレクトリに依存関係を保存することができる
- `go mod tidy`で`go.mod`から不要な依存関係を削除できる
  - 不用意な`go get`で依存関係が増えてしまうことがあるのでツールを`go get`したときはやったほうがよい

2019/02/16追記 `tidy`は「タイディ」と読むとのこと（Perlでも使われている）。
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">ピピーッ、tidy タイディ警察です。dan the interruptされるぞ！</p>&mdash; songmu (@songmu) <a href="https://twitter.com/songmu/status/1096368547146559493?ref_src=twsrc%5Etfw">2019年2月15日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


CIでの設定例は[ちまきん](https://twitter.com/__timakin__)さんや[だっく](https://twitter.com/duck8823)さんが公開してくれている。

- timakin/go-module | CircleCI Orbs Registory
  - https://circleci.com/orbs/registry/orb/timakin/go-module
- GitHub Actions のワークフローで Go 1.11 Modules のキャッシュを扱う | This is my life.
  - https://duck8823.hatenablog.com/entry/2018/12/31/090732

# Go 1.12の変更
Go1.12のリリースノートに記載されているModules関連の変更を確認しておく。
なお、全ての変更はmodule-aware mode（`GO111MODULE=on`）の場合である。

- Go 1.12 Release Notes
  - https://tip.golang.org/doc/go1.12

確認（1.11との比較）はGo1.11.5とGo1.12rc1のDockerイメージ上で同じコマンドを実行することで行なった。

- [[Go]未リリース版のGoの仕様や実際の動きを確認する #golangjp](/2019/02/14/investigate-go-unreleased-version/)

比較につかったDockerfileやスクリプトなどは以下のリポジトリにある。

- https://github.com/budougumi0617/gomodules-explore

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/gomodules-explore" data-iframely-url="//cdn.iframe.ly/XQI0aX5"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>



## モジュール管理外でも`go get`などができるようになった
1.11のときは`go.mod`がないディレクトリで`go get`をすると失敗していたが、1.12ではそれが解消された。
（ただ、`go.mod`があるディレクトリで行なうと`go get`で取得した情報を`go.mod`に追加してしまうので注意が必要）。


```bash
# Go1.11 at outside module and  GOPATH
$ GO111MODULE=on go get -u golang.org/x/lint/golint
go: cannot find main module; see 'go help modules'
make: *** [run111_go_get] Error 1
```

```bash
# Go1.12 at outside module and  GOPATH
$ GO111MODULE=on go get -u golang.org/x/lint/golint
go: finding golang.org/x/lint/golint latest
go: finding golang.org/x/lint latest
go: downloading golang.org/x/lint v0.0.0-20181217174547-8f45f776aaf1
go: extracting golang.org/x/lint v0.0.0-20181217174547-8f45f776aaf1
go: finding golang.org/x/tools/go/gcexportdata latest
go: finding golang.org/x/tools/go/ast/astutil latest
go: finding golang.org/x/tools/go latest
go: finding golang.org/x/tools/go/ast latest
go: finding golang.org/x/tools latest
go: downloading golang.org/x/tools v0.0.0-20190214204934-8dcb7
```

`go.mod`がなくてもどこでも`go run`ができるようになった。

```bash
# Go1.11 in moudle outside
$ GO111MODULE=on go run -v main.go
go: cannot find main module; see 'go help modules'
make: *** [run111_run_module_on_outside] Error 1
```

```bash
# Go1.12 in moudle outside
$ GO111MODULE=on go run -v main.go
Fetching https://gopkg.in/pressly/chi.v2?go-get=1
Parsing meta tags from https://gopkg.in/pressly/chi.v2?go-get=1 (status code 200)
get "gopkg.in/pressly/chi.v2": found meta tag get.metaImport{Prefix:"gopkg.in/pressly/chi.v2", VCS:"git", RepoRoot:"https://gopkg.in/pressly/chi.v2"} at https://gopkg.in/pressly/chi.v2?go-get=1
go: finding gopkg.in/pressly/chi.v2 v2.1.0
...
```

## `go env GOMOD`が`/dev/null`を報告する
カレントディレクトリで参照している`go.mod`ファイルを示す`GOMOD`変数がある。
module-aware modeの時、Go1.11では参照する`go.mod`ファイルがないと空白文字列を返していたが、Go1.12は`/dev/null`を返すようになった。
```bash
# Go1.11 with GO111MODULE=on
root@2d5cbf159edc:/tmp/test# GO111MODULE=on go env|grep GOMOD
GOMOD=""
```

```bash
# Go1.12 with GO111MODULE=on
root@a36b23164e3d:/tmp/test# GO111MODULE=on go env|grep GOMOD
GOMOD="/dev/null"
```

# 言語バージョンが保存される。
`go.mod`に`go`ディレクティブが追加され、そのモジュール内のファイルで使用されている言語バージョンを示すようになった。
`go.mod`に記載がない場合は、現在のリリースバージョンが設定される（1.12扱い）。

```bash
root@b83f6e3b9a00:/tmp/test# GO111MODULE=on go mod init test
go: creating new go.mod: module test
root@b83f6e3b9a00:/tmp/test# cat go.mod
module test

go 1.12
```

# `replace`でローカルのディレクトリのバージョン別に参照できるようになった
Go1.11でも`replace`ディレクティブを使ってルートディレクトリ以下のディレクトリに対して相対的に`replace`ディレクティブをおけるようになっていた。
Go1.12からは同じmodulesをバージョンごとに異なるディレクトリとして設定できるようになった。(発表後[@hajimehoshi](https://twitter.com/hajimehoshi)さんに指摘いただきました。ありがとうございます）。
バージョンの指定は`v0.0.0-00010101000000-000000000000`のようになる。例えば以下のような`go.mod`ファイルと適切なローカルディレクトリを用意しておく。

```
module example.com/m

replace (
	example.com/a => ./a
	example.com/a/b => ./b
)

replace (
	example.com/x => ./x
	example.com/x/v3 => ./v3
)

replace (
	example.com/y => ./y
	example.com/y/z/w => ./w
)

replace (
	example.com/v => ./v
	example.com/v v1.11.0 => ./v11
	example.com/v v1.12.0 => ./v12
)
```

Go1.11で上記の`go.mod`は`example.com/v`の部分のパースに失敗してビルドできない。

```bash
$ GO111MODULE=on go build -o test.exe
go: errors parsing go.mod:
/tmp/replace_import/go.mod:19: invalid module path
/tmp/replace_import/go.mod:20: invalid module path
/tmp/replace_import/go.mod:21: invalid module path
```

Go1.12ではビルドに成功し、`require`ディレクティブが自動生成される。

```
module example.com/m

replace (
	example.com/a => ./a
	example.com/a/b => ./b
)

replace (
	example.com/x => ./x
	example.com/x/v3 => ./v3
)

replace (
	example.com/y => ./y
	example.com/y/z/w => ./w
)

replace (
	example.com/v => ./v
	example.com/v v1.11.0 => ./v11
	example.com/v v1.12.0 => ./v12
)

require (
	example.com/a/b v0.0.0-00010101000000-000000000000
	example.com/v v1.12.0
	example.com/x/v3 v3.0.0-00010101000000-000000000000
	example.com/y v0.0.0-00010101000000-000000000000
)
```


# 今後の動向を知るためには
ModulesはGo1.13からはデフォルトで有効になる。1.13に向けて準備をしたいとき、最新の仕様を確認したいときは以下を確認する。
まず、計画されているModulesの2019年のプランは以下に記載されている。

- Go Modules in 2019
  - https://blog.golang.org/modules2019

また、未リリース版のGoの仕様は以下で確認できる。

- The Go Programming Language(tip)
  - https://tip.golang.org/doc/

Modulesの利用に関してissueなどでtipsが見つかったときは、以下のwikiの追加されている。

- Modules | golang/go wiki
  - https://github.com/golang/go/wiki/Modules

さっとbeta版やrc版を試したいときはdockerイメージを使ってみれば良いだろう。

- golang images | docker hub
  - https://hub.docker.com/_/golang


# 終わりに
depのファイルなどからGo Modulesへの移行は簡単に行える。
正直私はまだ`dep`を使っていたのだが、Go1.12を契機にそろそろModulesへの移行を始めようと思う（依存しているライブラリもちゃんと`go.mod`を配置していないと辛い模様）。
ただ、まだ理解できていないこともあるのでGo1.13までにはちゃんと使えるようになっておきたい。

# Go1.13以降のGo Modulesについて（2019/11/02追記）
Go1.13からはGo Modulesに加えてプロキシのエコシステムが加わった。また、`$GO111MODULE=auto`の挙動も若干変わっている。  
くわしくは[@ktr](https://twitter.com/ktr_0731)さんのブログ記事を読むといいだろう。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fsyfm.hatenablog.com%2Fentry%2F2019%2F08%2F10%2F170730" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>


# 参考
- Go & Versioning | research!rsc
  - https://research.swtch.com/vgo
- Go Modules in 2019
  - https://blog.golang.org/modules2019
- Modules | golang/go wiki
  - https://github.com/golang/go/wiki/Modules
- Goにおけるバージョン管理の必要性 − vgoについて −
  - https://www.slideshare.net/takuyaueda967/go-vgo-102442203
- モジュール対応モードへの移行を検討する
  - https://text.baldanders.info/golang/go-module-aware-mode/
- Understand Go Modules / Go Modulesを理解する
  - https://speakerdeck.com/linyows/go-moduleswoli-jie-suru?slide=21
- Semantic Versioning 2.0.0
  - https://semver.org/
