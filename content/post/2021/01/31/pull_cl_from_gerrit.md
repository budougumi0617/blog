+++
title= "[Go] まだレビュー中のgerrit上のコードパッチをローカルに取得する"
date= 2021-01-31T15:20:00+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","gotips","gerrit"]
keywords = ["Go","gerrit","CL"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

[@dice_zu][dice_zu]さんに教えてもらったことメモ。

<!--more-->

# TL;DR
- `golang.org/x/tools`のまだマージされていないコードをローカルで動かしたくなった
- gerritに直接作成されたパッチはGitHubにミラーされていない
- Change Listトップページの「DOWNLOAD」ボタンをクリックするとパッチを取得するいくつかの方法が提案される
    - `git fetch https://go.googlesource.com/go refs/changes/59/249759/2 && git checkout -b change-249759 FETCH_HEAD`
    - `git fetch https://go.googlesource.com/go refs/changes/59/249759/2 && git checkout FETCH_HEAD`
    - etc...

![コマンドリスト](/2021/01/31_cmd_list.png)


# gerrit上のコードパッチをローカルで実行したい
社内勉強会としてGoのコードリーティングを隔週で行なっている。  
昨年のアドベントカレンダーに同勉強会のことを書いたことがきっかけで[@dice_zu][dice_zu]さんや[@po3rin][po3rin]さんにもゲストとして参加していただいている。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fgo-code-reading" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

今週の勉強会では2021/01/30現在まだマージされていない`golang.org/x/tools`の次のChange List（CL）[^change_list]のコードを読むことになった。

- tools/go/analysis: add a new VET checker "DeadMutation"
    - https://go-review.googlesource.com/c/tools/+/287034

ここで、SSAの動きがコードを読むだけでは理解できなかったのでローカルでコードをいじりながらテストコードを実行してみたくなった。


[^change_list]: GitHubのPull Request相当

# GerritのCLをローカルに取得する
Goプロダクトのコードはすべて[go.googlesource.com/](https://go.googlesource.com/)上で開発が行われている[^mirror]。  
GitHubならばghコマンドを使って`gh pr checkout ${PR_NUMBER}`とするだけだろう。  
しかし、上記のCLはgerritに直接作成されたもので、GitHubのミラーリポジトリ上にPRとして存在しなかった。  
gerrit上にしかないパッチの取得方法がわからなかったので[@dice_zu][dice_zu]さんに教えてもらった。

[^mirror]: GitHub上にあるリポジトリはミラーリポジトリ

## googlesource.com上にあるリポジトリをcloneする
まず大前提として該当リポジトリを`git clone`しておく。  
GitHubにある[golang/tools](https://github.com/golang/tools)リポジトリはミラーリポジトリなので、googlesource.com上にあるリポジトリをcloneする。

```bash
# https://go-review.googlesource.com/admin/repos/tools
$ git clone "https://go.googlesource.com/tools"
```

## パッチを取得するgitコマンドを表示する
あとは該当CLのパッチを取得するためのgitコマンドを生成すればよい。  
CLのトップページのdescriptionなどの右下にある「DOWNLOAD」ボタンをクリックする。
![ダウンロードボタン](/2021/01/31_dl_btn.png)

そうするといくつかのパッチを取得するgitコマンドが表示される。

![コマンドリスト](/2021/01/31_cmd_list.png)

とくにこだわりがなかれば一番上のbranchを取得するコマンドを実行すればよいだろう。

```bash
$ git fetch https://go.googlesource.com/tools refs/changes/34/287034/5 \
&& git checkout -b change-287034 FETCH_HEAD
```

これでローカルにCLの変更を取得することができた。

```
$ git log --oneline -n 3
4ec2bca5 (HEAD -> change-287034) tools/go/analysis: add a new VET checker "DeadMutation"
514964b7 gopls/internal/hooks: improve license file test
68bf78a6 internal/lsp/cmd: improve help output of gopls subcommands
$ go test ./go/analysis/passes/deadmutation
ok      golang.org/x/tools/go/analysis/passes/deadmutation      0.072s
```

# 終わりに
今回このCLをコードリーディングの題材にしたのは[@dice_zu][dice_zu]さんの提案だった。  
私はマージされたコミットすら追えていないので、レビュー中のコードもしっかりウォッチされている[@dice_zu][dice_zu]さん流石という感じだった。

# 参考
- [私がGoのソースコードを読むときのTips](https://devblog.thebase.in/entry/go-code-reading)
- [tools/go/analysis: add a new VET checker "DeadMutation"](https://go-review.googlesource.com/c/tools/+/287034)


[dice_zu]: https://twitter.com/dice_zu
[po3rin]: https://twitter.com/po3rin