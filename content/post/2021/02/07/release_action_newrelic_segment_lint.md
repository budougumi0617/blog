+++
title= "[Go] 新規追加した関数にNew RelicのSegmentを入れ忘れていたら警告するGitHub Actionsを作った"
date= 2021-02-07T00:19:47+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go","newrelic","o11y","github"]
tags = ["golang","newrelic","actions","github"]
keywords = ["GitHub Actions","Go","linter","NewRelic"]
twitterImage = "logos/newrelic.png"
+++

Goのアプリで新規関数を含むPull Requset（PR）を作ったとき、New RelicのSegmentをいれてなかったら怒るreviewdogのGitHub Actionsを作った。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/action-newrelic-segment-lint" data-iframely-url="//cdn.iframe.ly/aLlaE2N?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- New RelicでGoのアプリケーションのSpanを取得するには、関数やメソッドごとにSegmentを作成する
- Segmentを作っていなかったら怒るActionsを作った
    - https://github.com/budougumi0617/action-newrelic-segment-lint
- 実際作った時にTipsなど

![実際のコメント](/2021/02/07_pr_comment.png)

GitHub MarketPlaceにも公開している。

https://github.com/marketplace/actions/action-newrelic-segment-lint

# Spanを計測していない関数を追加するPull Request（PR）はGitHub Checksで警告したい
New RelicでGoのアプリケーションのSpanを取得するには、関数やメソッドごとにSegmentsを明示的に埋め込んでいく必要がある。

```go
func SampleHandler(w http.ResponseWriter, req *http.Request) {
  defer newrelic.FromContext(req.Context()).StartSegment("sample_handler").End()
  fmt.Fprintf(w, "Hello, %q", req.URL.Path)
}
```

- [Instrument Go segments][igs]

以前、既存コードの関数やメソッドに対してFunctions Segmentを埋め込む`nrseg`コマンドを作った。

- [GoのアプリにNew Relic APMを導入する時とても便利なCLIを作った](/2021/01/17/release_nrseg/)

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/nrseg" data-iframely-url="//cdn.iframe.ly/HrUY585?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

しかし、普段開発しているサービスが完全にコードフリーズされていることは少ないだろう。  
我々は常にコードを変更したり、追加している。そのため、ある時点で存在するコードに対してだけでなく、**今後追加されていく関数やメソッドもSpanを計測しているか確認していく必要がある**。  
そのため、GitHub Acrionsとreviewdogを使って「PRで追加された関数がSegmentを埋め込んでいなかったらエラーを返す」Actionを作成した。

このようなYAMLを`.github/workflow`ディレクトリ以下に配置しておくだけで利用ができる。

```yaml
name: nrseg
on: [pull_request]
jobs:
  linter_name:
    name: runner / nrseg inspect
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: budougumi0617/action-newrelic-segment-lint@v0
        with:
          github_token: ${{ secrets.github_token }}
          # 無視したいディレクトリと対象ディレクトリ
          nrseg_flags: "-i testing ./src"
          reporter: github-pr-review
          level: warning
```

実際の挙動はサンプルPRを見てもらったほうが早いだろう。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/action-newrelic-segment-lint-examples/pull/1" data-iframely-url="//cdn.iframe.ly/J6KT0YP"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


上記PRのコメントのように新しく追加されたコードの中でSegmentを呼んでいないコードに対して警告をコメントする。

![実際のコメント](/2021/02/07_pr_comment.png)

これで新しく追加される関数にSegmentが正しく宣言されているかチェックできる体制が作れる。


# 作ったときのTips
機能説明は以上なので、以降は作ったときのメモなど。

## reviewdogを使ったActionsの作り方
PRで該当箇所にコメントするにはreviewdogを利用している。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/reviewdog/reviewdog" data-iframely-url="//cdn.iframe.ly/AlDOvej?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

reviewdogを使ったActionsはtemplateを使えばすぐ作れる。  
バージョンアップをサポートするActionや、Action内で使うツールの更新を検知するActionなども一緒に定義されていてとても便利。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/reviewdog/action-template" data-iframely-url="//cdn.iframe.ly/ykzZKkE?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## reviewdogに自作ツールの結果を読み込ませる
reviewdogが認識できる警告のフォーマットはREADMEの通り。

- https://github.com/reviewdog/reviewdog#input-format

今回はgolangci-lintと同じ出力形式でエラーを出力しているのでreviewdogがサポートしてくれる。  
注意点としてはエラー出力ではなく標準出力で解析結果を出力すること。

## Actionsの動作確認をしたいとき
- https://docs.github.com/ja/actions/managing-workflow-runs/enabling-debug-logging

実際に別のリポジトリでActionsを動かしてみると思ったとおりに実行されなかったりすることがある。  
そういうときは`ACTIONS_RUNNER_DEBUG`と`ACTIONS_STEP_DEBUG`というsecret定数をどちらも値を`true`で宣言する。
そうするとActionsの実行ログにデバッグ出力も表示されるようになる。

## GitHubにリリースされた圧縮ファイルを実行するワンライナー
Actions内で[`github.com/budougumi0617/nrseg`のリリースされているTarファイル](https://github.com/budougumi0617/nrseg/releases)を解凍して実行ファイルを使いたいとする。  
このような場合以下のような`cURL`コマンドを実行すると該当のTarファイルが取得できる。grepはUbuntuで実行可能なバイナリが入っている圧縮ファイル名前を確認すること。  
Goのコマンドならバイナリさえリリースされていれば、特別なコンテナを用意せずにActionsが書ける。

```bash
TEMP_PATH="$(mktemp -d)"
PATH="${TEMP_PATH}:$PATH"

curl -L "$(curl -Ls https://api.github.com/repos/budougumi0617/nrseg/releases/latest | grep -o -E "https://.+?_Linux_x86_64.tar.gz")" -o nrseg.tar.gz
tar -zxvf nrseg.tar.gz -C "${TEMP_PATH}" 
rm nrseg.tar.gz
```

# 終わりに
今回はじめてActionsを作ったが、reviewdogの周辺ツールがとてもリッチなのですぐ完了できた。  
「静的解析ツールを作る -> reviewdogでactionsにする」を速く開発できればどんどん自動コードレビューを手厚くできそう。

実際の関数の判定ロジックなどはnrsegリポジトリに実装してあるのだが、だいぶ汚いコードになってしまったので直しておきたい。

# 参考
- https://github.com/budougumi0617/action-newrelic-segment-lint
- https://github.com/marketplace/actions/action-newrelic-segment-lint
- [GoのアプリにNew Relic APMを導入する時とても便利なCLIを作った](/2021/01/17/release_nrseg/)
- https://docs.newrelic.com/docs/agents/go-agent/instrumentation/instrument-go-segments
- https://docs.github.com/ja/actions/managing-workflow-runs/enabling-debug-logging

[igs]: https://docs.newrelic.com/docs/agents/go-agent/instrumentation/instrument-go-segments
