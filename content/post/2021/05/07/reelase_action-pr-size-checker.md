+++
title= "テストコードなどは除外してからPRのサイズを警告するActionsを作った"
date= 2021-05-07T09:30:11+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["github","oss"]
tags = ["review","pr","github"]
keywords = ["GitHub","Pull Request"]
twitterImage = "logos/github.png"
+++

Pull Request（PR）の追加行数を計測して、指定行数以上だった場合はPRにコメントするGitHub Actionsをつくった。  
フィルターパターンを設定しておけば、テストコードや設定ファイルの追加行数を無視する。

https://github.com/budougumi0617/action-pr-size-checker

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/marketplace/actions/action-pr-size-checker" data-iframely-url="//cdn.iframe.ly/2rVGShx"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->


# TL;DR
- PRは小さい方がいい
    - https://blog.ploeh.dk/2015/01/15/10-tips-for-better-pull-requests/
    - 邦訳: https://yakst.com/ja/posts/1625
- GitHub ActionsでPRの追加行数を計測するツールを作成した
    - https://github.com/budougumi0617/action-pr-size-checker
- 設定したファイルパターンを無視するのが特徴
    - テストコードや設定ファイルならばいくら追加してもよいと考える


利用したいリポジトリで次のように `YOUR_REPO/.github/workflows/check_pr_size.yml` を作成するだけで利用できる。

```yaml
name: check-pr-size
on: [pull_request]
jobs:
  linter_name:
    name: runner / check-pr-size
    runs-on: ubuntu-latest
    steps:
      - uses: budougumi0617/action-pr-size-checker@v0
        with:
          github_token: ${{ secrets.github_token }}
          max_added_count: 300
          filter_pattern: "go.mod|go.sum|.*_test.go|.*.md|.*.golden|.*.yml"
```

2021/05/07時点では指定行数以上の変更を行なったPRには次のようなコメントが行われる。

<img src="/2021/05/07_gigi_report.png" alt="PRへのコメントイメージ" width="500">


# PRは小さい方がいい
流派や諸説あるとは思うが、PRは小さい方が良いと思っている。理由は以下のブログが説明してくれている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://blog.ploeh.dk/2015/01/15/10-tips-for-better-pull-requests/" data-iframely-url="//cdn.iframe.ly/E7JgqyT"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
日本語訳
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://yakst.com/ja/posts/1625" data-iframely-url="//cdn.iframe.ly/cKyyp92"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

記載以外の理由を挙げるとそもそもの方向性が間違っていた場合、デカイいPRは手戻りも大きいしマージするときの心理的負荷がレビューア・レビューイ双方にとって大きい。


# 既存のツールとの比較
すでにPRのサイズをチェックするツールは世の中にいくつか存在する。
有名なOSSでいうとDangerなどがPRのサイズをチェックしてくれる。

- https://github.com/danger/danger

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Ftech.connehito.com%2Fentry%2Fdanger" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

他にも、GitHub ActionsのMarketplaceで「PR size」などで検索するといくつかActionsが見つかる。
しかし、既存ツールにはひとつ懸念事項があり、今回自作した。

## テストを書く人ほど差分が増えてエラーになってしまう問題
PRのサイズを計測するときのデメリットとしては、テストコードの追加も考慮に入れてしまうと本末転倒な事態が発生してしまうことだ。  
TDDやレガシーコード改善ガイドの「テストコードがないコードはレガシーコードだ！」という言葉、テストの重要性が叫ばれている現在PRを作るときは一緒にテストコードを含めることがデファクトになっているはずだ。  
テストコードが含まれたPRで闇雲に追加行数でPRをリジェクトしてしまうと、「テストをしっかり書く人」が不幸になってしまう。
私はテストコードについては適切なないようならばどんどん追加してもう良いと思っている。よいテストコードは仕様を明らかにするため、テストコードを含んでいるほうがPRの理解しやすいという点もある。  

既存のツールをざっとみたところ「特定のファイルを無視してPRの行数を確認する」という機能はなさそうだったので、今回のツールを作成した。


# ファイルパターンで特定ファイルは無視するaction-pr-size-checker
今回作ったGitHub Actionsは前節の問題に対応するため、正規表現でファイルパターンを登録できる。
なので、テストコードやテストで利用する設定ファイル（ゴールデンファイル・YAML etc）を無視してPRの追加行数を確認する。

https://github.com/budougumi0617/action-pr-size-checker

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/marketplace/actions/action-pr-size-checker" data-iframely-url="//cdn.iframe.ly/2rVGShx"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

利用するときはリポジトリで次のように `YOUR_REPO/.github/workflows/check_pr_size.yml` を作成するだけで利用できる。  
閾値となる追加行数の上限は`max_added_count`、無視するファイルパターンを`filter_pattern`で指定するだけた。

```yaml
name: check-pr-size
on: [pull_request]
jobs:
  linter_name:
    name: runner / check-pr-size
    runs-on: ubuntu-latest
    steps:
      - uses: budougumi0617/action-pr-size-checker@v0
        with:
          github_token: ${{ secrets.github_token }}
          max_added_count: 300
          filter_pattern: "go.mod|go.sum|.*_test.go|.*.md|.*.golden|.*.yml"
```

上限を超えるPRはCheck Statusがレッドになり、解析結果が以下のようにコメントされる。

<img src="/2021/05/07_gigi_report.png" alt="PRへのコメントイメージ" width="500">

## ファイルパターンの正規表現
内部的にはGoで実装しているので、ファイルパターンはRE2形式の正規表現を利用できる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/google/re2" data-iframely-url="//cdn.iframe.ly/to1VyVj?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 中身の話
実装はGoのバイナリをシェルスクリプトで動かしているだけだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/gigi" data-iframely-url="//cdn.iframe.ly/Aytf2mH?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/action-pr-size-checker" data-iframely-url="//cdn.iframe.ly/PAqCOtF?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

GitHub ActionsはreviewdogのActionsテンプレートを流用している。
- https://github.com/reviewdog/action-template

ファイルごとの追加行数の計算にもreviewdogの実装をライブラリ的に利用している。

- https://pkg.go.dev/github.com/reviewdog/reviewdog@v0.11.0/diff

# 終わりに
reviewdogには普段からお世話になっているが、今回はコードでもお世話になってしまった。とても感謝。  
余裕があったらcheck statusの変更オプションや任意のコメントに変更できる機能をつけたい[^comment]。

[^comment]: 深夜のテンションで「[デカ過ぎんだろ...](https://dic.nicovideo.jp/a/%E3%83%87%E3%82%AB%E9%81%8E%E3%81%8E%E3%82%93%E3%81%A0%E3%82%8D...)」とコメントするように実装しようと思ったがさすがにやめた。

# 参考
- [10 tips for better Pull Requests by Mark Seemann](https://blog.ploeh.dk/2015/01/15/10-tips-for-better-pull-requests/)
- https://github.com/budougumi0617/action-pr-size-checker
- https://github.com/budougumi0617/gigi
- [Dangerで始めるPull Requestチェック自動化 - コネヒト開発者ブログ](https://tech.connehito.com/entry/danger)
- https://github.com/google/re2/wiki/Syntax
- https://github.com/reviewdog/action-template
- https://pkg.go.dev/github.com/reviewdog/reviewdog@v0.11.0/diff