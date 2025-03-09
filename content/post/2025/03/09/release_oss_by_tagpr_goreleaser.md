+++
title= "tagprとGoReleaserを使ったOSSの自動リリース"
date= 2025-03-09T15:00:00+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["github","go"]
tags = ["github","github actions","tagpr","goreleaser"]
keywords = ["github","github actions","tagpr","goreleaser"]
twitterImage = "twittercard.png"
+++

去年、3年ぶりくらいに自作のCLIツールをリリースしたけど、どうやってリリースしてたかすっかり忘れていた。  
CIは壊れてたし、せっかくなので`tagpr`と`GoReleaser`（と`GitHub App`）を使って2024年らしいデプロイパイプラインを作り直した。  
リリースPRをマージしたらHomebrewへのバイナリリリースまで自動化できて満足。ちょっと時間が経ってしまったけどメモを残しておく。

<!--more-->

# TL;DR
- GitHub Actionsで複数リポジトリを操作するときはGitHub Appを使った認証を使う
    - https://docs.github.com/ja/apps/creating-github-apps/authenticating-with-a-github-app
- `tagpr`を使うとGitHubリリース用のPRを作ってくれる
    - https://github.com/Songmu/tagpr
- `tagpr`と`GoReleaser`を使うと、GitHubリリースやHomebrewなどにビルドした結果も自動でリリースできる
    - https://goreleaser.com/
- リリース用のPRをマージすれば、自動でバイナリリリースまでするActionsを作成できた
    - https://github.com/budougumi0617/nrseg/blob/v0.0.10/.github/workflows/tag-and-release.yml

なお、この記事では自作ツールをHomebrew-tap対応する方法は触れない。

また、このあと説明で記載するYAMLは次のリポジトリで使われることを想定している。
- https://github.com/budougumi0617/nrseg
- https://github.com/budougumi0617/homebrew-tap

# 久しぶりにOSSをメンテすると何も覚えていない
去年、3年ほど放置していた自作OSSのCLIツールを久々にいくつかメンテした。
ツールのコードのメンテ自体はすぐできたけど、リリース手順を完全に忘れていた。  
放置前からGitHub Actionsを使ったバイナリリリースの設定などはしていたものの、GoReleaserのActionsをアップデートしたら壊れてしまったりしたので、令和のイケてるリリースフローを作るためにしっかり作り直すことにした。

# 自動化の方針
最終的に実現した流れは以下の通り。

- メインブランチにPRをマージすると、リリース用のPRが作成される
    - 次の修正PRをマージすると自動的にリリース用のPRが更新される
- メインブランチにリリース用のPRをマージすると、GitHub上でリリースタグが作成される
    - CHANGELOGも自動生成する
- リリースされたらGoReleasrがバイナリデータを作成し、GitHubとHomebrew-tap用のリポジトリにバイナリをアップロードする。
    - 具体的には、CLIツールのリポジトリへのバイナリアップロードとhomebrew-tap用のリポジトリを更新する

# やったこと
具体的には次の設定を行った。
- GitHub Appを使った認証トークンの発行準備
- `tagpr`の設定
- `tagpr`と`GoReleaser`を組み合わせたActionsフローの作成

## GitHub Appで認証情報を管理
Actions内で複数のリポジトリを操作するときは認証トークンの発行が必要になる。
GitHub Appを使えばトークンの有効期限を心配する必要もないようなので、こちらを利用することにした。

- https://docs.github.com/ja/apps/creating-github-apps/authenticating-with-a-github-app

GitHub Appを作成しておけば、この公式actionでActions内でトークンを利用できる。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/marketplace/actions/create-github-app-token" data-iframely-url="//iframely.net/Cd1w2ci"></a></div></div><script async src="//iframely.net/embed.js"></script>

具体的な手順は公式の手順書をみればできる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://docs-internal.github.com/ja/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow" data-iframely-url="//iframely.net/wkvLLVp"></a></div></div><script async src="//iframely.net/embed.js"></script>

今回実現するフローだと、以下のリソースに対してRead/Write権限が必要になる

- Code
    - これはデフォルト有効かも
- Pull Requets
- Contents
- Workflows

これを個人アカウントにインストールしておく。
あとは該当リポジトリのシークレットに秘密鍵を登録してActionsに`create-github-app-token`のステップを追加する。
後続のステップでは`steps.app-token.outputs.token`を参照することで認証が必要なGitHubの操作が可能になる。

```yaml
jobs:
  first-job-name:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/create-github-app-token@v1
        id: app-token
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.PRIVATE_KEY }}
      - name: Check out source code
        uses: actions/checkout@v4
        with:
          token: ${{ steps.app-token.outputs.token }}
```

### ハマったところ
GitHub App自体のPermissionを調整した後は、個人アカウント側で更新内容を受け入れる必要がある。

https://github.com/settings/installations

## 2. tagprを使ったリリースPR作成の自動化
[tagpr](https://github.com/Songmu/tagpr)は、バージョンタグの作成やリリース用のPRを自動で生成してくれるツールだ。

- Pushイベントをトリガーに、次のリリースタグの自動判定・PR作成を実施
    - ラベルを使うことでパッチバージョンアップ、メジャーバージョアップなどが可能
- リリース用PRマージすることで自動的に新しいリリースタグが設定される

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://songmu.jp/riji/entry/2022-09-05-tagpr.html" data-iframely-url="//iframely.net/cAjMAge?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/Songmu/tagpr" data-iframely-url="//iframely.net/f9vV0Y1?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

リポジトリのREADMEを読みながら設定すれば簡単にセットアップできた。ハマったところもなし。
CHANGELOGの自動生成もしてくれるし、めちゃくちゃ便利。

- https://github.com/budougumi0617/nrseg/blob/v0.0.10/CHANGELOG.md


`.tagpr`、リポジトリに追加した後は次のようなActionsを作成すればよい。
```yaml
name: tag-and-release

on:
  push:
    branches:
      - master
permissions:
  contents: write
  pull-requests: write

jobs:
  tagpr:
    runs-on: ubuntu-latest
    outputs:
      tagpr-tag: ${{ steps.run-tagpr.outputs.tag }}
    steps:
      - uses: actions/create-github-app-token@v1
        id: app-token
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.PRIVATE_KEY }}
      - name: Check out source code
        uses: actions/checkout@v4
        with:
          token: ${{ steps.app-token.outputs.token }}
      - id: run-tagpr
        name: Run tagpr
        uses: Songmu/tagpr@v1
        env:
          GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
```

バイナリを作成する必要がないライブラリ系のリポジトリならばここで終わりでもよい。


## 3. GoReleaserで自動リリース
`GoReleaser`はGo（Rustもできるらしい）で書かれたソフトウェアのリリースプロセスを自動化してくれるツール。

- クロスコンパイル対応もしてくれる
- Homebrewタップへの反映なども自動化できる

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://goreleaser.com/" data-iframely-url="//iframely.net/zkcbliz?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

こちらは`.goreleaser.yml`を作成したあとは、以下のような`job`を定義すれば、GitHub Appで取得した認証情報を使ってバイナリをリリースできる。

```yaml
  goreleaser:
    needs: tagpr
    if: needs.tagpr.outputs.tagpr-tag != ''
    runs-on: ubuntu-latest
    steps:
      - uses: actions/create-github-app-token@v1
        id: app-token
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.PRIVATE_KEY }}
          owner: "budougumi0617"
          repositories: |
            nrseg
            homebrew-tap
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ steps.app-token.outputs.token }}
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version-file: go.mod
          cache: true
      - name: Run GoReleaser
        uses: goreleaser/goreleaser-action@v6
        with:
          distribution: goreleaser
          version: "latest"
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
```

年のため、GitHub Appに作業できるリポジトリを明示的に指定している。

```yaml
          repositories: |
            nrseg
            homebrew-tap
```

### ハマったポイント
今回メンテしたリポジトリでリリースしたバイナリを使ったActionsを書いていたので、リリースされるファイルパスなどの調整でちょっとハマった。
まっさらな状態からリリースするならばそんなにハマらないと思う。

## 最終的なGitHub ActionsとActionsで作成されるもの

ここまでの設定をすべてまとめると、次のようなActionsになる。

https://github.com/budougumi0617/nrseg/blob/v0.0.10/.goreleaser.yml
```yaml
name: tag-and-release

on:
  push:
    branches:
      - master
permissions:
  contents: write
  pull-requests: write

jobs:
  tagpr:
    runs-on: ubuntu-latest
    outputs:
      tagpr-tag: ${{ steps.run-tagpr.outputs.tag }}
    steps:
      - uses: actions/create-github-app-token@v1
        id: app-token
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.PRIVATE_KEY }}
      - name: Check out source code
        uses: actions/checkout@v4
        with:
          token: ${{ steps.app-token.outputs.token }}
      - id: run-tagpr
        name: Run tagpr
        uses: Songmu/tagpr@v1
        env:
          GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
  goreleaser:
    needs: tagpr
    if: needs.tagpr.outputs.tagpr-tag != ''
    runs-on: ubuntu-latest
    steps:
      - uses: actions/create-github-app-token@v1
        id: app-token
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.PRIVATE_KEY }}
          owner: "budougumi0617"
          repositories: |
            nrseg
            homebrew-tap
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ steps.app-token.outputs.token }}
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version-file: go.mod
          cache: true
      - name: Run GoReleaser
        uses: goreleaser/goreleaser-action@v6
        with:
          # either 'goreleaser' (default) or 'goreleaser-pro'
          distribution: goreleaser
          # 'latest', 'nightly', or a semver
          version: "latest"
          args: release --clean
        env:
          # need to access other repository for brew-tap
          GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
```

このActionsを設定した後、リポジトリで修正PRをマージすると、次のようなリリース用PRが作成される。

- https://github.com/budougumi0617/nrseg/pull/35

このPRをマージすると、リリースタグが作成され、バイナリも配置される

- https://github.com/budougumi0617/nrseg/releases/tag/v0.0.10

CHANGELOGも自動生成される。うれしい。

- https://github.com/budougumi0617/nrseg/blob/v0.0.10/CHANGELOG.md

# おわりに
GitHub Actionsを活用することで、リリース作業をほぼ完全に自動化できた。
自分のOSSを作るときはこれをテンプレにしたい（最近は作れていないけれど…）。  
数年放置していたけれど、`go.mod`が入った後だったしGoのコードは問題なく動くのも確認できた。よい。  
久しぶりにちゃんとActionsを自分で書いたら認証トークンの払い出し方とか変わっていて、（PAT払い出してシークレットに設定して終わりだった）ちょっと前は放牧的な時代だったんだなーと思った。


# 参考
- GitHub App による認証
    - https://docs.github.com/ja/apps/creating-github-apps/authenticating-with-a-github-app
- tagpr
    - https://github.com/Songmu/tagpr
- GoReleaser
    - https://goreleaser.com/
- GitHub Appsトークン解体新書：GitHub ActionsからPATを駆逐する技術
    - https://zenn.dev/tmknom/articles/github-apps-token
- tagprで実現するPull Request上で進めるOSSのリリースマネジメント
    - https://k1low.hatenablog.com/entry/2022/10/04/083000

