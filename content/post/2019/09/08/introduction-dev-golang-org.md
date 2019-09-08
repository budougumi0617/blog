+++
title= "Go本体の開発進捗を確認できるGo Development Dashboard"
date= 2019-09-08T10:31:15+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang"]
keywords = ["Go", "Golang"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


Goの開発状況を確認する`Go Development Dashboard`を見つけたのでメモしておく。

<!--more-->

# TL;DR
- Goの開発はGitHubとGerritをもちいて行われている
    - ソースコードは https://go.googlesource.com/go
        - [GitHub上のリポジトリ][golanggo]はミラー
    - Issueの管理はGitHub上で行われている
        - https://github.com/golang/go/issues
    - 変更（Pull Request相当）のレビューはGerrit上で行われている
        - https://go-review.googlesource.com/q/status:open
- GitHubのIssueとGerrit上のCL(Pull Request)をシームレスに見れるのが`Go Development Dashboard`
    - https://dev.golang.org/

[golanggo]: https://github.com/golang/go

# Go本体の開発では何が使われているのか
Go本体のソースコードはGitHubで確認することができる。しかし、GitHub上のリポジトリはミラーリポジトリだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/go" data-iframely-url="//cdn.iframe.ly/ELYEmIC"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

オリジナルリポジトリは`googlesource.com`に存在する。

https://go.googlesource.com/go

（最近はGitHubフローの手順も合わせて紹介されているが、）コントリビューションガイドも`googlesource.com`を`git clone`するように記載されている。

- Contribution Guide
    - https://golang.org/doc/contribute.html

GitHubでいうPull Request（以下PR）を使ったレビューの流れをどのようにやっているかというと、Gerritが使われいる。
変更に対するレビューのやりとりは、（GerritのPR相当である）Change List(以下CL)で行われており、以下のURLから確認することができる。

- StatusがOpenのCL一覧
    - https://go-review.googlesource.com/q/status:open


ただ、Issue管理についてはGitHub上で行われており、マイルストーンなども設定されている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/go" data-iframely-url="//cdn.iframe.ly/ss03w4d"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# Go Development Dashboard
変更理由となるIssueはGitHub上、変更内容はGerrit上、となると、2つのWebサイトを見比べないといけない。
今まで不便だなと感じてたが、実は両者を一括して確認できるダッシュボードがあった。

- Go Development Dashboard
    - https://dev.golang.org/

飾りっ気も何もないダッシュボードだが、このダッシュボードを使うと各リリースマイルストーンを簡単に一覧することができる。

- Release dashboard
    - https://dev.golang.org/release

この画面ではGitHubのIssueと、それに紐づくCLが確認できるので、（クリックして別ウインドウで開くのは各サイトだが、）GitHubとGerritを行き来する必要がない。

![releases list](/2019/09/08_releases.png)

また、またStatusがOpenなCLも一覧・検索することができる。

- open changes
    - https://dev.golang.org/reviews


# 終わりに
今まであまりGo本体の変更を追えていなかったが、このダッシュボードがあれば効率的に確認できそうだ。
また、Go本体への変更はすべてCIが実行されている。そのCIの結果を一覧できる`Go Build Dashboard`もあるようだ。

- Go Build Dashboard
    - https://build.golang.org/

![CI status](/2019/09/08_build.png)

見てるだけではしょうがないので自分もCL作れないかIssueを色々見てみよう。

# 参考
- [Go Development Dashboard](https://dev.golang.org/)
- [Go Build Dashboard](https://build.golang.org/)
- [Contribution Guide](https://golang.org/doc/contribute.html)
