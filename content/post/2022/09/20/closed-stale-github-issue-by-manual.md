+++
title= "GitHub CLIを使って任意の条件のissueを一括closeする（一括操作する）"
date= 2022-09-20T09:36:26+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["github","project"]
tags = ["github","cli","issue","project management"]
keywords = ["生産性","github cli","GitHub","GitHub issue"]
twitterImage = "logos/github.png"
+++

業務のタスク管理としてGitHub Project（Issue）を使っている。  
Issueの棚卸しとして、古いIssueを一括closeしたときの操作をまとめる。  
[GitHub GraphQL API v4](https://docs.github.com/ja/graphql)を用いてやろうと思ったが、GitHub CLIを利用する方法が一番カンタンだった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://cli.github.com/" data-iframely-url="//iframely.net/Yqle0ig?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

<!--more-->

# TL;DR
- 古いGitHub Issueを一括closeしたくなった。
- GitHub CLIを使うとページングなどを考えずにissue検索をWeb UI同様にできる
    - https://cli.github.com
- closeしたいissueのURLを取得したら`gh issue xxx`コマンドを使って任意の操作ができる
- APIリミットには注意する
- いちど整理したら`actions/stale Action`を使えばよい
    - https://docs.github.com/en/actions/managing-issues-and-pull-requests/closing-inactive-issues

なお、この記事で利用しているGitHub CLIのバージョンは次の通り。

```bash
$ gh version
gh version 2.15.0 (2022-09-06)
https://github.com/cli/cli/releases/tag/v2.15.0
```

# GitHubのissueをまとめて操作したい（closeしたい）
業務ではタスク管理にGitHub Project（Issue）を使っている。  
「とりあえず起票しておくか」で結局着手していないIssueが溜まってきていたので古いissueを一括でcloseすることにした。  
後述する`actions/stale Action`を設定して待つのも良いのだが、まず一括closeするissueのリストを作っておきたかったのでGitHub CLIを使うことにした。

# GitHub CLI（`gh`コマンド）でGitHubのissueを整理する
issueを整理するにはGitHub CLIがとても便利だった。最初「GitHub API v4（GraphQL）でやるか、わからんな」と思っていたが、GitHub CLIならばローカルでもトライアンドエラーがしやすく、すぐできた。  
GitHub CLIは各GitHubの操作を行える`hub`コマンドの後進で、実際にCLIから利用するときは`gh`コマンドとなる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://cli.github.com/" data-iframely-url="//iframely.net/Yqle0ig?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

やることの流れは以下のとおりだ。

- `gh issue list`コマンドで記録用に一括closeするGitHub Issueをリストにしておく
    - `--jq` オプションを使えばMarkdownのリストもすぐつくれる
- 該当のGitHub IssueのURL一覧を取得する
- GitHub IssueのURLを使って`gh issue close`コマンドで1つ1つissueをcloseしていく。

GitHub CLIの`gh issue`コマンドでできる操作ならば、close以外の一括操作も可能だ。
https://cli.github.com/manual/gh_issue

なお、ここから実行する`gh`コマンドはすべてローカルにcloneした該当のリポジトリのディレクトリ内で実行するものとする。

# 記録用に一括closeするGitHub Issueをリストにしておく
正当な理由でcloseされたGitHub Issueと区別するため、まずは一括closeするGitHub Issueのリストを作っておく。

## GitHub CLIでIssueを検索するときの条件
GitHub CLIで一括closeしたいGitHub Issueを検索（フィルター）しないといけない。  
`gh`コマンドでGitHub Issueを検索するときは、Web UIと同じ条件でフィルターすることができる。
まずはWeb UI上で一括closeしたいGitHub Issueを検索する。

https://github.com/${ORG_NAME}/${REPO_NAME}/issues

私は「今年に入って1度も更新されていないGitHub Issue」を一括closeしようと思ったので次のフィルター条件になった。

```
is:issue is:open sort:update-asc updated:<2022-01-01
```

## 一括closeするGitHub IssueのリストをMarkdownで作る。
フィルター条件がわかったらそれを使って記録用の一覧を作る。  
一覧はMarkdownで作るので、`gh`コマンドの出力もMarkdown形式で出力する。`gh`コマンドは単独で`--jq`オプションを使って出力を加工できるのでこれを使えばMarkdownの表に利用できる出力が得られる。

```shell
# 実行零では--limit 3にしている。見込まれるissueの数で--limitオプションも調整する。
$ gh issue list --limit 3 --json number,title,url,author \
--jq '.[] | "| [#" + (.number|tostring) + " " + .title + " by " + .author.login + "]("
 + .url + ")|"' \
--search "is:issue is:open sort:update-asc updated:<2022-01-01"

| [#2 20210423 Gophers Code Reading Party by budougumi0617](https://github.com/basebank/gophers-code-reading-party/issues/2)|
| [#3 20210513 Gophers Code Reading Party by budougumi0617](https://github.com/basebank/gophers-code-reading-party/issues/3)|
| [#1 20210405 Gophers Code Reading Party by hgsgtk](https://github.com/basebank/gophers-code-reading-party/issues/1)|
```

表のヘッダーとなるMarkdownの下にコピペすれば次のような一覧が作成できる。

|一括closeリスト|
|---|
| [#2 20210423 Gophers Code Reading Party by budougumi0617](https://github.com/basebank/gophers-code-reading-party/issues/2)|
| [#3 20210513 Gophers Code Reading Party by budougumi0617](https://github.com/basebank/gophers-code-reading-party/issues/3)|
| [#1 20210405 Gophers Code Reading Party by hgsgtk](https://github.com/basebank/gophers-code-reading-party/issues/1)|

一括closeする前にチームメンバーへ一覧を共有してcloseしていいか確認しておくとよいだろう。

# GitHub Issueを一括closeする
close候補のGitHub Issueの一覧を共有し、チームで合意がとれたら実際にcloseする。

## GitHub Issueをcloseしていく`gh`コマンドのワンライナー

`gh`コマンドを利用するとSQLでいうN+1のAPIコールになってしまうが次のコマンドで愚直に一括closeできる。
正確にいうと逐次closeだが…。

```shell
$ gh issue list --limit 500 --json number,title,url,author \
--jq '.[] | .url' --search "is:issue is:open sort:update-asc updated:<2022-01-01" \
 | while read URL; do
  gh issue close ${URL} -c "2022年issue棚卸し ref: ${作成した一覧のURL}"
done
```

やっていることは、`gh issue list`コマンドとフィルター条件で出力したcloseしたいGitHub IssueのURLを出力している。  
その後、そのGitHub Issueの各URLに対して`gh issue close`コマンドを使ってcloseするAPIを実行しているだけだ。  
GitHub Issueをcloseする際にコメントを残すことができるので、未解決のまま一括closeされたことをわかりやすくするためのコメントを残している。
各GitHub Issueに対してAPIを実行し続けるので数によってはターミナルを開きっぱなしにして他の作業をして待機することになる。

いきなりcloseさせるのが怖い場合は`gh issue view`コマンドなどで一度動作確認しておくとよい。  
なお、SlackなどにGitHub Issueの変更情報を通知している場合は一時的にオフにしておかないと大量の「closeしました」「コメントがありました」の通知が流れることになる。
## APIのレートリミットに注意すること
GitHub CLIも結局はGitHub APIのラッパーでしかないのでAPIコールのレートリミットに気をつける必要がある。
あまりに大量のissueをcloseすることになりそうなときは事前に確認し、必要に応じてsleepコマンドなどをはさみながら実行すること。

GitHub APIのレートリミットを確認するには次のコマンドを実行すればよい。
```bash
$ gh api rate_limit --jq '.resources.core'
{"limit":5000,"remaining":4979,"reset":1662484504,"used":21}

$ date -r "$(gh api rate_limit --jq '.resources.core.reset')"
Wed Sep  7 02:15:04 JST 2022
```

以下のzennを参考にしたが、macOSでは少し`date`コマンドのオプションが異なるようだ。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://zenn.dev/hankei6km/scraps/4f02c89052a62c" data-iframely-url="//iframely.net/38Y5JG7"></a></div></div><script async src="//iframely.net/embed.js"></script>

# いちど整理をしたらstalek Actionsを設定する
定期的にこのような操作を行なうより、`actions/stale Action`を設定するほうが効率的だ。  
一度GitHub Issueの整理ができたらあとは`actions/stale Action`を利用して一定期間未更新のGitHub Issueは自動closeするようにしておこう。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/marketplace/actions/close-stale-issues" data-iframely-url="//iframely.net/yyyfTVO"></a></div></div><script async src="//iframely.net/embed.js"></script>

使い方はGitHubの公式ドキュメントを読むと良いだろう。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://ghdocs-prod.azurewebsites.net/en/actions/managing-issues-and-pull-requests/closing-inactive-issues" data-iframely-url="//iframely.net/aR2nDS9"></a></div></div><script async src="//iframely.net/embed.js"></script>

# 終わりに
最初は「GitHub API v4使うことになりそうだしまずはGraphQL調べるところかな」なんて思って手が進んでいなかったが、GitHub CLIを利用すればすぐ終わった。  
「ずっと放置されているけれど、やるかやらないで言ったら絶対やったほうがよい」タスクを今回思い切って全部closeした。  
GitHub Issue（GitHub Issue）以外だったらもっとめんどくさかったかもしれないので、普段から慣れ親しんでいるGitHubでタスク管理していてよかった。  
今後は`actions/stale Action`もいい感じに設定して活用していきたい。
# 参考
- https://cli.github.com
    - https://cli.github.com/manual/gh_issue_close
    - https://cli.github.com/manual/gh_issue_list
- https://docs.github.com/en/actions/managing-issues-and-pull-requests/closing-inactive-issues
- https://github.com/marketplace/actions/close-stale-issues