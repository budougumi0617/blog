+++
title= "reviewdogを使ったGitHub Actionsの作り方"
date= 2021-02-16T09:00:00+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go","github"]
tags = ["golang","reviewdog","github"]
keywords = ["Go","reviewdog","GitHub","GitHub Actions"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

reviewdogを使ったGitHub Actionsを作る過程をメモしておく。

- [[Go] 新規追加した関数にNew RelicのSegmentを入れ忘れていたら警告するGitHub Actionsを作った](/2021/02/07/release_action_newrelic_segment_lint/)

<!--more-->

# TL;DR
- template repositoryを使うと簡単にreviewdogのGitHub Actionsが作れる
    - https://github.com/reviewdog/action-template"
- Goのlinterを作る場合は以下の2つのどちらかを使っていればreviewdog対応が簡単
    - https://pkg.go.dev/golang.org/x/tools/go/analysis
    - https://pkg.go.dev/go/token#Position

# 自作linter+reviewdogでGitHub Actionsを作りたい
CIでlinterを実行し、問題があったときは非ゼロでExitするだけでもいいのだが、reviewdogを使えばPRにコメントを残したり、GitHub checksに対応することができる。  
効率的なレビューを実施するためにlinterによるチェックをreviewdogを使ったGitHub Actionsにしておいたほうが良いだろう。

先日、実際にreviewdogを使ったGitHub Actionsを作ったが、とても簡単にできたのでその作成方法をメモしておく。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/action-newrelic-segment-lint" data-iframely-url="//cdn.iframe.ly/aLlaE2N"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 事前準備: 自作linterで実装しておくべきポイント
まずいくつかのポイントを抑えて自作linterを作っておくとreviewdog対応がとても簡単になる。

## バイナリファイルをダウンロードできるようにしておく
GitHubリリースページにバイナリが配布されていると簡単にActions内から実行できる。  
Goの場合はGo Releaserなどを利用してリリースしておく。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://goreleaser.com/" data-iframely-url="//cdn.iframe.ly/zkcbliz?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## 標準出力へエラーフォーマットどおりに結果を出力する
reviewdogは設定さえすれば任意の出力フォーマットのエラー結果出力をパースすることができる。

- https://github.com/reviewdog/reviewdog#input-format

いくつかのメジャーなlinterのエラー出力は事前定義されている。  
Goの場合は、golintやgolangci-lintと同じフォーマットでエラーを出力するようにしておくとよい。
golintのフォーマットは後述する方法で簡単に実装できる。

もし別の事情でエラーフォーマットが変更できないときでも、プレイグラウンドでフォーマットを検証しながら設定を考えることができる。

- https://reviewdog.github.io/errorformat-playground/

# 実際の作成手順
お作法を守ったlinterを作っておけば、Actionsを作ること自体は簡単だ。


## templateを利用する
reviewdogのGitHub Actionsはtemplate repositoryが用意されているので、まずはこれを使ってrepositoryを初期化すればよい。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/reviewdog/action-template" data-iframely-url="//cdn.iframe.ly/ykzZKkE"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Goの場合はDockerを作らずともシェルスクリプトだけでActionsの実装ができるので、reviewdog/action-golangci-lintのファイルを移植しても良いと思う。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/reviewdog/action-golangci-lint" data-iframely-url="//cdn.iframe.ly/se1bvdE"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## スクリプトを書く
linterを実行するだけならば、Actionsの中で実際に行なうことは次のステップだけだ。

- ツールをダウンロードする
- linterを実行してreviewdogにパースしてもらう

GitHubでバイナリをリリースしているならば、次のようなスクリプトだけでActions内でlinterが実行できるようになる。

- https://github.com/budougumi0617/action-newrelic-segment-lint/blob/master/script.sh

```baash
TEMP_PATH="$(mktemp -d)"
PATH="${TEMP_PATH}:$PATH"

curl -L "$(curl -Ls https://api.github.com/repos/budougumi0617/nrseg/releases/latest | grep -o -E "https://.+?_Linux_x86_64.tar.gz")" -o nrseg.tar.gz
tar -zxvf nrseg.tar.gz -C "${TEMP_PATH}"
rm nrseg.tar.gz
```

あとはGitHubトークンを設定してreviewdogと一緒に実行すればよい。


```bash
nrseg inspect ${INPUT_NRSEG_FLAGS} \
  | reviewdog -f=golint \
      -name="${INPUT_TOOL_NAME}" \
      -reporter="${INPUT_REPORTER:-github-pr-check}" \
      -filter-mode="${INPUT_FILTER_MODE:-added}" \
      -fail-on-error="${INPUT_FAIL_ON_ERROR:-false}" \
      -level="${INPUT_LEVEL}" \
      ${INPUT_REVIEWDOG_FLAGS}
```

これと設定ファイルであるaction.ymlを清書すれば終わりだ。

- https://github.com/budougumi0617/action-newrelic-segment-lint/blob/master/action.yml

ここまでやっておくと、GitHubが自動的にリポジトリをGitHub Actions用のリポジトリであることを認識してくれる。  
あとはリリースを行うと、リリースページ編集時に自動的に「Publish this Action to the GitHub Marketplace」などの操作が追加で行える。

- [GitHub Marketplaceでのアクションの公開](https://docs.github.com/ja/actions/creating-actions/publishing-actions-in-github-marketplace)

# reviewdogでパースできるlint結果の作り方
reviewdogでPRに含まれるコードの特定行にコメントを残すにはlintエラーに位置情報を付与して出力する必要がある。

```bash
# main.goの24行目にlint errorを発見したとき出力
main.go:24:1: Server.Shutdown no insert segment
```

Goの場合、reviewdogが`-f=golint`オプションでパースできる情報を簡単に作ることができる。


## golang.org/x/tools/go/analysis pkgを利用する
golang.org/x/tools/go/analysis pkgを利用して作成されたlinterはgolintなどと同じフォーマットでエラーを出力する。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/tools" data-iframely-url="//cdn.iframe.ly/T71XGV2"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

以前作った自作ツールの結果を例にすると、とくに意識せずともファイル名と該当エラー行の行数情報などが出力される。

- [golang.org/x/tools/go/analysisでLinterツールを自作する #gounco #golang](/2019/02/01/how-to-use-analisys-package/)

```bash
go vet -vettool=$(which regexponce) ./testdata/src/a
# github.com/budougumi0617/regexponce/testdata/src/a
testdata/src/a/a.go:16:30: regexp.Compile must be called only once at initialize
testdata/src/a/a.go:21:34: regexp.MustCompile must be called only once at initialize
testdata/src/a/a.go:27:6: regexp.MustCompile must be called only once at initialize
testdata/src/a/a.go:31:34: regexp.MustCompile must be called only once at initialize
testdata/src/a/a.go:36:31: regexp.MustCompile must be called only once at initialize
testdata/src/a/a.go:38:32: regexp.MustCompile must be called only once at initialize
testdata/src/a/a.go:43:31: regexp.MustCompile must be called only once at initialize
testdata/src/a/a.go:45:32: regexp.MustCompile must be called only once at initialize
```

## token.Positionを使う
- https://pkg.go.dev/go/token#Position

golang.org/x/tools/go/analysis pkgを利用していなくても、 `*token.FileSet` と `token.Pos` オブジェクトがあれば同様の出力を簡単に取得することができる。  
静的解析をしていれば上記の情報は簡単にとれるのでこれを組み合わせると必要情報を所定フォーマットで出力できる。

```go
p := fs.File(pos).Position(pos)
fmt.Printf("%s:%d:%d: lint error message\n", p.Filename, p.Line, p.Column)
```

# その他
- https://github.com/reviewdog/action-template/tree/master/.github/workflows

templateに入っているGitHub ActionsはGitHub Actions自体のメンテに便利なCI定義ばかりなのでぜひ使っておいたほうが良い。

- ラベルをつけてPRをマージすると自動で所定のセマンティックバージョンをインクリメントしたリリースをつくるrelease.yml
- 依存ツールがバージョンアップされたことを検知し、GitHub Actions内のバージョン設定の更新PRを作成するdepup.yml

などが同梱されている。

# 参考
- https://github.com/reviewdog/action-template
- https://github.com/reviewdog/action-golangci-lint
- https://reviewdog.github.io/errorformat-playground/
- [GitHub Marketplaceでのアクションの公開](https://docs.github.com/ja/actions/creating-actions/publishing-actions-in-github-marketplace)
- https://pkg.go.dev/golang.org/x/tools/go/analysis
- https://pkg.go.dev/go/token#Position
- [golang.org/x/tools/go/analysisでLinterツールを自作する #gounco #golang](/2019/02/01/how-to-use-analisys-package/)
- [[Go] 新規追加した関数にNew RelicのSegmentを入れ忘れていたら警告するGitHub Actionsを作った](/2021/02/07/release_action_newrelic_segment_lint/)