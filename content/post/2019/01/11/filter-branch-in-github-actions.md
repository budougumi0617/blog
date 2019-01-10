+++
title= "GitHub Actionsで特定のブランチのときのみワークフローを実行する #github #actions #ci"
date= 2019-01-11T07:17:52+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["github"]
tags = ["github","actions","ci"]
keywords = ["github","actions","ci"]
twitterImage = "logos/github.png"
+++

2019/01現在Public BataのGitHub ActionsでCircleCIの`branches only`相当の処理を出来ないか調査した。

- GitHub Actions
  - https://github.com/features/actions/

![Failed](/2019/01/11-failed.png)

<!--more-->

# TL;DR
- 特定のブランチ名のときだけ動くGitHub Actionsを定義したい
- Workflowに対する設定はないが、Actionで設定することができる
- actions/bin/filterを使う
  - https://github.com/actions/bin/tree/master/filter

サンプルブランチは以下。

- budougumi0617/actions-filter-branch
  - https://github.com/budougumi0617/actions-filter-branch

なお、この記事は2019/01/11時点で最新のGitHub Actionsの情報になる。（2018/12/18リリースのPublic BETA版）

- Enabling Actions on Public Repositories
  - https://developer.github.com/actions/changes/#--enabling-actions-on-public-repositories

# GitHub Actionsとは
ここではGitHub Actionsの詳細は省くが、GitHub Actionsは現在Public BETA版が公開されているGitHub公式のワークフローの自動化サービスだ。

- GitHub Actions： みなさんが開発し、GitHubで実行
  - https://blog.github.com/jp/2018-10-24-action-demos/

まだ利用できない方は以下のページの`Sign up for the beta`ボタンをクリックしておけばいずれリポジトリに「Actions」タブが着くようになる

- GitHub Actions
  - https://github.com/features/actions/

GitHub Actionsでは、一連の処理のまとまりをworkflow、各処理はActionsと定義している。各Actionsはそれぞれで「別の」コンテナを起動して実行される。コンセプトはGCPのCloud Buildに近い。
workflowの定義は`/.github/main.workflow`ファイルをリポジトリに配置することで設定でき、GitHub上で該当ファイルを開くとGUIエディタからも編集できる。

# やりたいこと
CI/CDを設定していると、例えば、`master`ブランチが更新されたときだけ実行されるデプロイジョブなどを作っていないだろうか。
CircleCIでいうと、`branches only`相当の処理になる。

- branches | Configuring CircleCI - CircleCI
  - https://circleci.com/docs/2.0/configuration-reference/#branches

これをGitHub Actionsでも設定できるのか調べた。

# actions/bin/filterを使う
実は公式ですでに用意されているActions用のコンテナがあるのでそれを使えば簡単に実現できた。

- https://github.com/actions/bin/tree/master/filter

GUIエディタからも選ぶことができる。

![Filters for GitHub Actions](/2019/01/11-filter.png)

今回は`master`ブランチのときのみ後続の`echo`コマンドのActionを実行する簡単なWorkflowを定義してみた。
`actions/bin/filter`コンテナで実行するActionを定義し、スクリプトとフィルターしたいブランチ名を引数に設定しておく。

```
action "Filters for GitHub Actions" {
  uses = "actions/bin/filter@master"
  args = "branch master"
}
```

なお、任意の`shell`コマンドを実行するActionはGUIエディタからすぐ選べないので、自分で`actions/bin/sh@master`を選んで定義する。
workflow全体は以下のようになる。

- https://github.com/budougumi0617/actions-filter-branch/blob/master/.github/main.workflow

```
workflow "filter branch" {
  on = "push"
  resolves = ["Shell"]
}

action "Filters for GitHub Actions" {
  uses = "actions/bin/filter@master"
  args = "branch master"
}

action "Shell" {
  uses = "actions/bin/sh@master"
  needs = ["Filters for GitHub Actions"]
  args = ["echo success"]
}
```

![work flow](/2019/01/11-workflow.png)

このworkflowはリポジトリにpushイベントが発生されるたびに実行される。

- https://github.com/budougumi0617/actions-filter-branch/actions

実行した結果が以下だ。`master`ブランチのときは最後までWorkflowが実行される。

- https://github.com/budougumi0617/actions-filter-branch/actions/workflow-runs/MDEwOkNoZWNrU3VpdGU0NzU0NTk5Mw==

![Succeeded](/2019/01/11-succeeded.png)

`master`ブランチでないときは`Neutral`というstatusでWorkflowは最後まで実行されなかったので、意図通りブランチ名でActionsを制御できた。

- https://github.com/budougumi0617/actions-filter-branch/actions/workflow-runs/MDEwOkNoZWNrU3VpdGU0NzU1MDMxNg==

![Failed](/2019/01/11-failed.png)


# actions/bin/filterで行なっていること
これだけだと寂しいので、`actions/bin/filter`の中も確認してみた。
リポジトリの中を見るといくつかの簡単なスクリプトが用意されており、READMEに書かれている通りに使えばよい。

- https://github.com/actions/bin/blob/master/filter/README.md
- https://github.com/actions/bin/tree/master/filter/bin
  - action
  - branch
  - label
  - ref
  - tag
  - user

スクリプトの中身も非常にシンプルで、`branch`スクリプトは単に`ref`スクリプトへのエイリアスだ。`ref`スクリプト自身も簡単なcase文が書かれているだけだった。

- https://github.com/actions/bin/blob/master/filter/bin/ref

```bash
#!/bin/sh

set -e

pattern=$1

case "$GITHUB_REF" in
  "")
    echo "\$GITHUB_REF is not set"
    exit 1
    ;;
  $pattern)
    echo "$GITHUB_REF matches $pattern"
    exit 0
    ;;
  *)
    echo "$GITHUB_REF does not match $pattern"
    exit 78
esac
```

`$GITHUB_REF`はActions実行毎に設定されている環境変数で、Actions実行時のブランチ名やタグ名が取得できる。

- Environment variables | GitHub Actions
  - https://developer.github.com/actions/creating-github-actions/accessing-the-runtime-environment/#environment-variables

> GITHUB_REF
>
> The branch or tag ref that triggered the workflow. For example, refs/heads/feature-branch-1. If neither a branch or tag is available for the event type, the variable will not exist.

# 終わりに
今回はTwitterで[@teitei_k](https://twitter.com/teitei_tk)さんと話題に上がったので他の確認よりまず先にやってみた。`$GITHUB_REF`が取れるところまでは知っていたので、スクリプトを自作しようと思ったがすでに公式で提供済みのものがあった。

Beta版だからなのかそういう設計思想のなのか（おろらく後者だが）、GitHub ActionsではCircleCIなどほどとリッチなYAMLのような設定を書けない。
ただ、複雑な設定を書き始めると属人化しはじめたりするし、YAML地獄になったりしがちだ。私がGoのシンプルなところが好きなせいなのもあるかもしれないが、ここのActionsとして小さく設定する今の塩梅が好きだ。実行ステータスを表示できるバッジなど早くほしいのと、publicリポジトリはまだpush通知をトリガーにしかできないので、PRのイベントにも対応されると嬉しい。


# 参考
- budougumi0617/actions-filter-branch
  - https://github.com/budougumi0617/actions-filter-branch
- GitHub Actions： みなさんが開発し、GitHubで実行
  - https://blog.github.com/jp/2018-10-24-action-demos/
- GitHub Actions
  - https://github.com/features/actions/
- actions/bin
  - https://github.com/actions/bin/
- Environment variables | GitHub Actions
  - https://developer.github.com/actions/creating-github-actions/accessing-the-runtime-environment/#environment-variables
