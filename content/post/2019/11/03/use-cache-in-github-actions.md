+++
title= "[Go] GitHub Actionsでキャッシュ機能を使う #github"
date= 2019-11-03T18:04:33+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go", "github"]
tags = ["golang", "github", "actions", "ci"]
keywords = ["Go", "golang", "GitHub", "GitHub Actions", "Actions", "ビルドキャッシュ"]
twitterImage = "logos/github.png"
+++

GitHub Actionsで待望のキャッシュ機能が使えるようになった。  
Windowsコンテナでジョブを実行していると少しハマる感じだったが、自分のGoのリポジトリのGitHub Actionsでキャッシュを使えるようになった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/actions/cache" data-iframely-url="//cdn.iframe.ly/vNWhS6o"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- GitHub Actionsでついにキャッシュが使えるようになった
  - https://github.com/actions/cache
- 言語別の利用方法もExamplesに記載されている
  - https://github.com/actions/cache/blob/master/examples.md
- Windowsコンテナ上で実行する場合は2019/11/03現在工夫が必要
  - https://github.com/actions/cache/issues/39
- 45秒かかっていたジョブの実行が26秒になったので効果も確認できた

自分のGoのリポジトリでGitHub Actionsのキャッシュを使う変更をしたPRは以下の通り。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/layer/pull/5" data-iframely-url="//cdn.iframe.ly/dlanAo4"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

```yaml
    - name: Use Cache(on Windows)
      if: runner.os == 'Windows'
      uses: actions/cache@preview
      with:
        path: ~/go/pkg/mod
        # 2019/11/03現在、Windowsの場合PATHを変えないと失敗する
        key: ${{ runner.os }}-go-${{ hashFiles('**\go.sum') }}
        restore-keys: |
          ${{ runner.os }}-go-
    - name: Use Cache
      if: runner.os != 'Windows'
      uses: actions/cache@preview
      with:
        path: ~/go/pkg/mod
        key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
        restore-keys: |
          ${{ runner.os }}-go-
    - name: Download Modules
      if: steps.cache.outputs.cache-hit != 'true'
      run: go mod download
```

# GitHub Actionsでキャッシュが使えるようになった
2019年11月ほどからGitHub Actionsでキャッシュが利用できるようになった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/actions/cache" data-iframely-url="//cdn.iframe.ly/vNWhS6o"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

キャッシュは前回のジョブの実行で取得した依存ライブラリなどを次回以降のジョブでも再利用できる機能だ。
この機能があると依存ライブラリのダウンロードにかかる時間を省略することができ、ジョブの実行時間短縮につながる。
CircleCIでは以前からこのキャッシュ機能が利用可能だった。

- 依存関係のキャッシュ | CircleCI
  - https://circleci.com/docs/ja/2.0/caching/

# 言語別の利用方法もExamplesに記載されている
各言語でキャッシュをどのように定義すればいいかは以下のリポジトリにあるExamplesを見ればよい。  
どの言語でも、依存関係のlockファイルと保管場所のPATHがわかっていれば利用可能だ。

- Examples
  - https://github.com/actions/cache/blob/master/examples.md

Goの場合は以下のようになるだろう。

```yaml
    - name: Use Cache
      if: runner.os != 'Windows'
      uses: actions/cache@preview
      with:
        path: ~/go/pkg/mod
        key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
        restore-keys: |
          ${{ runner.os }}-go-
    - name: Download Modules
      if: steps.cache.outputs.cache-hit != 'true'
      run: go mod download
```


# Windows上で実行した際にちゃんと動かない
MacOSやLinuxコンテナ上でジョブを実行している場合は上記で定義は完了する。ただ、Windowsコンテナでジョブを実行している場合は一工夫必要だった。
サンプルコードの定義だと、以下のIssueのように、ファイルを探索する際のPATHがWindows上では正しく解釈できないようだ。

```bash
##[error]The template is not valid. 'hashFiles(**/go.sum)' failed. \
Search pattern '**/go.sum' doesn't match any file under 'd:\a\layer\layer'

```
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/actions/cache/issues/39" data-iframely-url="//cdn.iframe.ly/dqojS6a"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Windows用の定義は以下のようになる。

```yaml
    - name: Use Cache(on Windows)
      if: runner.os == 'Windows'
      uses: actions/cache@preview
      with:
        path: ~/go/pkg/mod
        # 2019/11/03現在、Windowsの場合PATHを変えないと失敗する
        key: ${{ runner.os }}-go-${{ hashFiles('**\go.sum') }}
        restore-keys: |
          ${{ runner.os }}-go-
```

# キャッシュを使ってみて
実際に自分のGoのリポジトリでGitHub Actionsのキャッシュを使ってみた。
変更の差分は以下のPRにまとまっている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/layer/pull/5" data-iframely-url="//cdn.iframe.ly/dlanAo4"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

キャッシュなしのビルド出力の抜粋は以下だ。ジョブ全体では45秒かかっていた。

- [キャッシュなしで実行したActions](https://github.com/budougumi0617/layer/commit/ce8304ed1b233a7d9c2fb2b400bc246d7ca39372/checks?check_suite_id=293461218)

```bash
Download Modules 14s
go: finding golang.org/x/sys v0.0.0-20190215142949-d0b11bdaac8a
Run go mod download
go: finding github.com/google/go-cmp v0.3.1
go: finding golang.org/x/tools v0.0.0-20191014205221-18e3458ac98b
go: finding golang.org/x/net v0.0.0-20190620200207-3b0461eec859
go: finding golang.org/x/sync v0.0.0-20190423024810-112230192c58
go: finding golang.org/x/xerrors v0.0.0-20190717185122-a985d3407aa7
go: finding golang.org/x/crypto v0.0.0-20190308221718-c2843e01d9a2
go: finding golang.org/x/text v0.3.0
go: finding golang.org/x/sys v0.0.0-20190215142949-d0b11bdaac8a
```

キャッシュありのビルド出力は以下になった。キャッシュが効いているので`go mod download`の時間は0秒になっている。
ビルド全体の実行時間も26秒になった。今回試したライブラリの依存先は大した量ではないので、もっと業務で利用するようなリポジトリでは大幅な時間短縮も見込めそうだ。

- [キャッシュありで実行したActions](https://github.com/budougumi0617/layer/commit/567ee885708004e31f0cadc1b657e369139ecad9/checks?check_suite_id=293465615)

```
-> Use Cache3s
46cbda8d3c8af50c3bea54f7ecbcfc8c  /home/runner/work/_temp/01ce8b69-3693-4872-b78b-6280fc624b81/cache.tgz
Run actions/cache@preview
/bin/tar -xz -f /home/runner/work/_temp/01ce8b69-3693-4872-b78b-6280fc624b81/cache.tgz -C /home/runner/go/pkg/mod
::set-output name=cache-hit,::true
Cache restored from key:Linux-go-bb075a445e8e16bc463699292dda306f5ec705d1d4e56e5a7ea406f3c3809685
Cache Checksum:
md5sum /home/runner/work/_temp/01ce8b69-3693-4872-b78b-6280fc624b81/cache.tgz
46cbda8d3c8af50c3bea54f7ecbcfc8c  /home/runner/work/_temp/01ce8b69-3693-4872-b78b-6280fc624b81/cache.tgz

-> Download Modules0s
Run go mod download
```

# 終わりに
今回は待望のGitHub Actionsのキャッシュ機能を試してみた。GitHub Actionsの感想でよく「キャッシュがちゃんとできればなあ」と言われていたので早くGAしてほしい。  
~~GitHub Actions本体はちょっと前にメールでGAと着ていた気がするのだが、Webのヘルプページなどを見るとまだpublic beta状態のようだ。早く正式にならないかな？~~

- About GitHub Actions
  - https://help.github.com/en/github/automating-your-workflow-with-github-actions/about-github-actions

11/13にGAらしい！いまのところPublicリポジトリは無料だしPrivateリポジトリもそんなに高くなさそう。楽しみだ。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">そのメールには11月13日にGA予定と書いてありましたね。料金はリンク先のPricingを見てね、というようなことが書いてあったような。<br>// 僕もGA楽しみにしてます！Self-Hosted Runnerも早く来てほしい。</p>&mdash; Yusuke KUOKA (@mumoshu) <a href="https://twitter.com/mumoshu/status/1190961371929661441?ref_src=twsrc%5Etfw">November 3, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 参考
- https://github.com/actions/cache
- [hashFiles() does not work for valid patterns | actions/cache #39](https://github.com/actions/cache/issues/39)
- [Use Cache in GitHub Actions | budougumi0617/layer #5](https://github.com/budougumi0617/layer/pull/5)

