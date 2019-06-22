+++
title= "textlinti/reviewdogで文書校正エラーをGitHubのプルリクエストにコメントする 2019年6月版"
date= 2019-06-22T11:03:47+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["ci", "review", "circleci"]
tags = ["circleci", "reviedog", "textlint", "review"]
keywords = ["技術書典", "CircleCI", "reviewdog", "textlint", "Re:VIEW"]
twitterImage = "twittercard.png"
+++


技術書典7に向けて、執筆用のRe:VIEWのリポジトリの準備を始めた。  
今回はCircleCIを使って文書校正を行なう設定を行なった。  
検索して出てくる情報は古い情報が多かったので2019年6月時点の設定方法をまとめる。

<!--more-->

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://techbookfest.org/event/tbf07" data-iframely-url="//cdn.iframe.ly/yaQdGlH?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# TL;DR
- Re:VIEW用の設定をしてPDFがビルドできる状態にする。
    - https://github.com/TechBooster/ReVIEW-Template/tree/master
- 文書の校正を行う`textlint`の設定を行う
- CircleCI上で`reviewdog`を使って`texlint`の結果をPull Request（PR）にコメントする
    - v0.9.12からインストール方法が変更されている。

サンプルリポジトリと実際に`reviewdog`経由で`textlint`の指摘をコメントしたPRは以下になる。

- https://github.com/budougumi0617/shoten-plate

- Add textlint, and check reviewdog
    - https://github.com/budougumi0617/shoten-plate/pull/1

# 同人誌作成もCIを使って自動化したい
私は今まで2回`golang.tokyo`として技術書典に参加していた。  
前回まではGitHub上で原稿（Re:VIEWファイル）を管理し、寄稿メンバー同士でPRを作って相互レビューをしていた。  
CI的な設定はせずにもろもろを手作業で行なっていたが、今回はなるべく自動化したいと考えた。  
そのため、まずは校正を自動で確認できるように`textlint`を使った自動化を試みた。

# Re:VIEWのリポジトリを作成する
まずは新規のRe:VIEWリポジトリを作成する。ここから最新版の構築済みRe:VIEW構成を取得する。

- https://github.com/TechBooster/ReVIEW-Template/tree/master

サンプルのPDFファイル以外を使わせてもらう。

# texlintrcを設定してローカルで文書校正できることを確認する
`textlint`は種々の静的解析ツールのように文書に対して文法や用語の使い方をチェックするツールだ。  
プラグインを使えば、Re:VIEWの文書ファイルにも適用することができる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://efcl.info/2015/09/10/introduce-textlint/" data-iframely-url="//cdn.iframe.ly/YlOR4Ea"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

まずは、CIの設定をする前にローカルで実行を確かめる。今回の設定は以下の記事を参考にした。

<iframe src="https://hatenablog-parts.com/embed?url=http%3A%2F%2Fblog.naoshihoshi.com%2Fentry%2F2018%2F10%2F15%2F113000" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no" allow="autoplay; encrypted-media"></iframe>

設定と言っても、`npm`コマンドで必要ライブラリをインストールしたあと、`.textlintrc`ファイルを作成して校正ルールを設定すればよい。  
どんなルールのオプションライブラリがあるのかは、GitHubのWikiを見る。

- https://github.com/textlint/textlint/wiki/Collection-of-textlint-rule#rules-japanese

今回はRe:VIEWで`textlint`を使うための`textlint-plugin-review`と技術書を書くための基本的なライブラリを使う。  
用語の表記ゆれのチェックを行う`textlint-rule-prh`ライブラリは明示的にインストールしなくても使えるようだ（？）。

```bash
$ npm install --save-dev npm install textlint \
    textlint-rule-preset-ja-technical-writing \
    textlint-plugin-review
```

次に設定ファイルを作成する。リポジトリルートに`.textlintrc`ファイルを以下のような内容で作成した。  
`textlint-rule-preset-ja-technical-writing`は複数の校正ルールを内包しており、個別にルールのON/OFFを設定できる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing" data-iframely-url="//cdn.iframe.ly/wGpDKUR"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

```json
{
    "rules": {
        "preset-ja-technical-writing": {
            "no-exclamation-question-mark": false,
        },
        "prh": {
            "rulePaths": [
                "./prh-rules/media/WEB+DB_PRESS.yml"
            ]
        }
    },
    "plugins": [
      "review"
    ]
}
```

`.textlintrc`ファイルを作成したあとは、適当な`sample.re`ファイルを`articles`ディレクトリに作成し`textlint`コマンドを実行してみる。

```bash
$ ./node_modules/.bin/textlint articles/sample.re

/Users/budougumi0617/go/src/github.com/budougumi0617/shoten-plate/articles/sample.re
   3:4   ✓ error  はじめ => 始め                                                                                                                                                prh
  10:15  ✓ error  することができ => でき                                                                                                                                        prh
  10:15  ✓ error  "することができるはず"は冗長な表現です。"することが"を省き簡潔な表現にすると文章が明瞭になります。参考: http://qiita.com/takahi-i/items/a93dc2ff42af6b93f6e0  ja-technical-writing/ja-no-redundant-expression
  12:1   ✓ error  DockerHub => Docker Hub                                                                                                                                       prh
  12:25  ✓ error  ツイッター => Twitter                                                                                                                                         prh

✖ 5 problems (5 errors, 0 warnings)
✓ 5 fixable problems.
Try to run: $ textlint --fix [file]
```

意図通りに文書の表記ゆれなどが指摘された。

# reviewdogとCircleCIを使ってPRに対してtextlintの検出エラーをコメントするようにする。
ローカルで実行が確認できたので、GitHubでPRを作成した時に、`textlint`のエラーをコメントで指摘するようにする。  
`reviewdog`は各種静的解析ツールの結果をコードレビューコメントとしてPRなどにコメントしてくれるツールだ。`textlint`にも対応している。

- reviewdog — A code review dog who keeps your codebase healthy
  - https://medium.com/@haya14busa/reviewdog-a-code-review-dog-who-keeps-your-codebase-healthy-d957c471938b

`reviewdog`とCircleCIを使ってCIを構築する。`reviewdog`の使い方はREADMEの該当機能の説明を読めばよい。

- Reporter: GitHub PullRequest review comment (-reporter=github-pr-review)
    - https://github.com/reviewdog/reviewdog#reporter-github-pullrequest-review-comment--reportergithub-pr-review

利用するバージョンは2019/06/22現在最新のv0.9.12を使う。

- v0.9.12: Installation method change / First step to org development ready release
    - https://github.com/reviewdog/reviewdog/releases/tag/v0.9.12

CircleCIの設定は次のブログ記事を参考にした。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://blog.kokuyouwind.com/archives/1597" data-iframely-url="//cdn.iframe.ly/08z3e1d"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

ただしこの記事は少し古く、最新の`reviewdog`の設定方法とは異なる。`reviewdog`のv0.9.12を使ってCircleCIの設定を行なうと次のようになる。

```yaml
version: 2
jobs:
  build:
    docker:
      - image: vvakame/review:3.1
        environment:
          REVIEWDOG_VERSION: 0.9.12
    steps:
      - checkout
      - restore_cache:
          keys:
            - npm-cache-{{ checksum "package-lock.json" }}
      - run:
          name: Setup
          command: npm install
      - save_cache:
          key: npm-cache-{{ checksum "package-lock.json" }}
          paths:
            - ./node_modules
      - run:
          name: install reviewdog
          command: "curl -sfL https://raw.githubusercontent.com/reviewdog/reviewdog/master/install.sh| sh -s  v$REVIEWDOG_VERSION"
      - run:
          name: lint
          command: "$(npm bin)/textlint -f checkstyle articles/*.re | tee check_result"
      - run:
          name: reviewdog
          command: >
              if [ -n "$REVIEWDOG_GITHUB_API_TOKEN" ]; then
                cat check_result | ./bin/reviewdog -f=checkstyle -name=textlint -reporter=github-pr-review
              fi
          when: on_fail
      - run:
          name: Build PDF
          command: npm run pdf
      - store_artifacts:
          path: ./articles/shoten7.pdf
          destination: shoten7.pdf
```

あとはCircleCI上の設定をいくつかすればよい。

# CircleCI上でCIを行なう準備をする

CircleCIで行う設定は、（対象のリポジトリへのビルドをONにするのは当然として、）`reviewdog`がPRへコメントをするときに利用するGitHubトークンの設定と、`Only build pull requests`の設定だ。

## reviewdogがPRへコメントできるようにCircleCiにGitHubトークンを設定する
`reviewdog`がPRへ`textlint`の検出エラーをコメントするために必要な認証情報をCirccleCIの環境変数に登録しておく。

- Personal access tokens
  - https://github.com/settings/tokens
- Reporter: GitHub PullRequest review comment (-reporter=github-pr-review)
  - https://github.com/reviewdog/reviewdog#reporter-github-pullrequest-review-comment--reportergithub-pr-review

Public/Privateリポジトリで必要な権限は異なるので、READMEをよく読むこと。

> Go to https://github.com/settings/tokens and generate new API token.
> Check repo for private repositories or public_repo for public repositories.

GitHubでトークンを払い出したら、`REVIEWDOG_GITHUB_API_TOKEN`という名前でCircleCIの環境変数にしておく。

- Environment Variables | CircleCI
    - `https://circleci.com/gh/YOUR_ACCOUNT/YOUR_REPO/edit#env-vars`

## `Only build pull requests`設定を有効にしておく
PRを作ったときにCircleCIが反応するようにしておかないと、コミットをプッシュした時点でCircleCIが反応し、`reviewdog`がうまく動かない。  
PRとデフォルトブランチへのプッシュにのみ反応するようにCircleCIの`Only build pull requests`設定を有効にしておく。

- Advanced Settings > Only build pull requests
    - https://circleci.com/docs/2.0/oss/#only-build-pull-requests

- Advanced Settings | CircleCI
    - `https://circleci.com/gh/YOUR_ACCOUNT/YOUR_REPO/edit#advanced-settings`


以上の設定をすべて完了して作成したPRが次のPRだ。

- Add textlint, and check reviewdog
    - https://github.com/budougumi0617/shoten-plate/pull/1

意図通り、PRに`reviewdog`から`textlint`で検出したエラーがコメントされた。

![reviewdog_comment](/2019/06/22_reviewdog_comment.png)

# 終わりに
他の方の記事を真似して設定すればすぐ終わると思っていたが、一部の環境構築の方法が最近変更されていたようなので、2019年版の設定方法をまとめた。  
`reviewdog`のコメント用に払い出したトークンが自分のアカウントなので、自分が指摘しているようになってしまったがここはしょうがないかなと思う。  
`textlint`の設定はまだ動作確認状態なので、もう少しチューニングしておきたい。あとはPRが作成されたらそのビルド結果のPDFをGoogleDriveに自動アップロードするCIを設定しておきたい。

# 参考
- [github.com/TechBooster/ReVIEW-Template/](https://github.com/TechBooster/ReVIEW-Template/tree/master)
- [github.com/reviewdog/reviewdog](https://github.com/reviewdog/reviewdog)
- [reviewdog — A code review dog who keeps your codebase healthy](https://medium.com/@haya14busa/reviewdog-a-code-review-dog-who-keeps-your-codebase-healthy-d957c471938b)
- [技術書を支える技術 | 黒曜の吹き溜まり](https://blog.kokuyouwind.com/archives/1597/)
- [textlintで日本語の文章をチェックする](https://efcl.info/2015/09/10/introduce-textlint/)
- [Re:VIEWで書いた文章の校正をCircleCIとtextlintでGitHubのPRに自動コメントする仕組み](http://blog.naoshihoshi.com/entry/2018/10/15/113000)

