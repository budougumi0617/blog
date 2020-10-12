+++
title= "goreleaserとGitHub Actionsを使えばGoのCLIはgit tagをpushするだけでGitHubとHomeBrewに自動リリースできる"
date= 2020-10-07T12:16:30+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","homebrew","cli"]
keywords = ["Go","homebrew","cli"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

goreleaserを使うとGo製のCLIのGitHubのリリースページの更新とHomeBrew Tap用リリースも簡単に行える。  
さらにGitHub Actionsを使えばYAMLを2ファイル追加するだけでgit tagに合わせて全自動リリースが可能になる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/goreleaser/goreleaser-action" data-iframely-url="//cdn.iframe.ly/N4iA3Tu?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- goreleaserはGoのバイナリをよしなにリリースできるツール
    - https://goreleaser.com/
    - GitHubのリリースページにリリースできる
    - HomeBrew Tapにもリリースできる
- GitHub Actionsも公式で提供されている
- Actionsを使えばgit tagをpushするだけで自動リリースできる


# GoのCLIを自動リリースしたい
GoでCLIのインストール方法は（Macの場合）主に次の3つになるだろう。

- `go get`でインストールする
- GitHubのリリースページなどにあるバイナリをインストールする
- HomeBrewでインストールする

goreleaserを使うと、GitHubリリースページへのバイナリアップロードやHomeBrew用の設定ファイルの作成がとても簡単になる。

- https://goreleaser.com/

# git tagに合わせてgoreleaser-actionを使って自動リリースをする。
GitHubへの自動リリースの手順はほとんど次の記事で説明されている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://tellme.tokyo/post/2020/02/04/release-go-cli-tool/" data-iframely-url="//cdn.iframe.ly/s3QQGTb"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

これに加えて、HomeBrew Tap用の設定をすればローカルに何も用意せず、HomeBrew Tap用リポジトリのRubyのファイルも自動生成・自動更新される。

## 必要なもの
自動リリースに必要なのは次の通り。
- リリースするCLIが入ったGitHubリポジトリ
- もろもろの設定をしたYAMLファイル
- HomeBrew Tap用のリポジトリ
    - USER_NAME/homebrew-tapというリポジトリをひとつ作っておくのが一般的
    - https://github.com/budougumi0617/homebrew-tap
- public repoへの書き込み権限を付与したGitHub Personal Token
    - https://github.com/goreleaser/goreleaser-action#limitation

## goreleaserの設定YAMLを用意する
まずはリリースの設定をしたYAMLファイルを用意する。  
公式ページを確認すればよいが、ほぼコピペで問題ない。

- https://goreleaser.com/customization/

以下のYAMLは`leetgode`というCLIをリリースする際に利用しているYAMLファイルだ。

- https://github.com/budougumi0617/leetgode/blob/v0.0.1/.goreleaser.yml


```yaml
before:
  hooks:
    - go mod tidy # tidyしておけば間違いない
builds:
  -
    main: ./cmd/leetgode # main.goファイルがある場所。
    env:
      - CGO_ENABLED=0
archives:
  - replacements:
      darwin: Darwin
      linux: Linux
      windows: Windows
      386: i386
      amd64: x86_64
checksum:
  name_template: 'checksums.txt'
snapshot:
  name_template: "{{ .Tag }}-next"
changelog:
  sort: asc
  filters:
    exclude:
      - '^docs:'
      - '^test:'
```

HomeBrew Tap用の設定は以下のとおり。これを`.goreleaser.yml`に追加しておく。  
いろいろ設定したいならば公式情報を参照すればよいがこちらもほぼコピペで大丈夫。

- https://goreleaser.com/customization/homebrew/

```yaml
brews:
  -
    name: leetgode # CLIの名前でよい
    github: # HomeBrew Tapをリリースするリポジトリ
      owner: budougumi0617
      name: homebrew-tap
    # ここは決め打ちで良い
    url_template: "https://github.com/budougumi0617/leetgode/releases/download/{{ .Tag }}/{{ .ArtifactName }}"
    commit_author: # homebrew-tapにcommitするときに使うGitHubアカウント
      name: goreleaserbot
      email: goreleaser@carlosbecker.com
    homepage: "https://budougumi0617.github.io/"
    description: "LeetCode CLI for Gophers."     # 適当なdescription
    test: | # リリースビルド後の動作確認用のコマンド。--versionやhelpなどを指定すればよいだろう
      system "#{bin}/leetgode help"
    install: |  # Goならばバイナリ名でOK
      bin.install "leetgode"
```

## GitHub Actionsの設定をする。
あとはGitHub Actionsを設定しておけば、自動リリースができる。

- https://goreleaser.com/ci/actions/

次の内容のYAMLファイルを`.github/workflows/goreleaser.yml`に設置する。  
こちらの参考YAMLも`leetgode` CLIで使っているものだ。

- https://github.com/budougumi0617/leetgode/blob/v0.0.1/.github/workflows/goreleaser.yml


```yaml
name: goreleaser

on:
  push:
    tags: #vX.X.Xというタグのときにリリースする
      - "v[0-9]+.[0-9]+.[0-9]+"
jobs:
  goreleaser:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0
      -
        name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.15
      -
        name: Run GoReleaser
        uses: goreleaser/goreleaser-action@v2
        with:
          version: latest
          args: release --rm-dist
        env:
          # need to access other repository for brew-tap
          GITHUB_TOKEN: ${{ secrets.GH_PAT }}
```

HomeBrew Tap用のリポジトリへの書き込み権限が必要なため、トークンを作っておかないといけない点だけ注意する。  
上記の例では、リポジトリのシークレットに`GH_PAT`という名前で払い出したトークンを設定している。  
やりかたは次のページを参照のこと。

- https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token
- https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets

## 動作確認
ここまで設定すればあとはすべて自動で実行される。  
tagを設定してリモートへプッシュすると、

```bash
$ git tag -a v0.0.1 -m "first release"
$ git push origin v0.0.1
```

GitHub Actionsが実行される。

- https://github.com/budougumi0617/leetgode/actions/runs/280533890

Actionsが実行されると、GitHub Releaseが作成される。

- https://github.com/budougumi0617/leetgode/releases/tag/v0.0.1

また、HomeBrew Tap用のリポジトリにRubyの設定ファイルが自動追加される。

- https://github.com/budougumi0617/homebrew-tap/blob/master/leetgode.rb

ローカルで`brew`コマンドを実行すると、リリースしたバイナリがインストールできるようになっている。

```bash
$ brew install budougumi0617/tap/leetgode
```


# 終わりに
GitHub Actions用のリポジトリができていたのでリリース作業がめちゃくちゃ簡単になった！  
そして設定もYAMLをコピペするだけでいいのでめちゃくちゃ簡単。  
CLIツールはちょくちょくつくるしmacOSユーザなので自分がラクするためにも積極的に使っていきたい。

# 参考
- https://github.com/goreleaser/goreleaser-action
- https://goreleaser.com/
- https://goreleaser.com/customization/homebrew/
- https://tellme.tokyo/post/2020/02/04/release-go-cli-tool/

