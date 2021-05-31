+++
title= "GitHub API v3でPrivateリポジトリのPull Requestのdiffを取得する"
date= 2021-05-31T09:30:50+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["github"]
tags = ["github","api"]
keywords = ["go-github","GitHub","pull request"]
twitterImage = "logos/github.png"
+++


GitHub API v3でPull Requestの情報をごにょごにょしようとしたらハマったのでメモ。  
GitHub API v3のPull RequestのJSONに含まれる`diff_url`は役に立たない。`Accept`ヘッダーでレスポンスが変化する。


<!--more-->

# TL;DR
- GitHub API v3でPull Requestの情報をdiff情報を取得したい
- `Get a pull request` APIのレスポンスの中にdiffのURLが存在する
    - このURLはprivateリポジトリの場合認証がとれず失敗する
- Acceptヘッダーを付けて`Get a pull request` APIを実行すればdiffを取得できる
    - https://docs.github.com/en/rest/overview/media-types#commits-commit-comparison-and-pull-requests


サンプルコードは以下。

```go
hc = oauth2.NewClient(ctx, oauth2.StaticTokenSource(
    &oauth2.Token{AccessToken: cfg.GitHubToken},
))

cli := github.NewClient(hc)
pr, _, _ := cli.PullRequests.Get(ctx, cfg.Owner, cfg.Repository, cfg.PullRequestNumber)
url := pr.GetURL()

req, _ := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
req.Header.Add("Accept", "application/vnd.github.v3.diff")

resp, err := hc.Do(req)
```


# Pull Requestのdiffを取得したい

GitHub API v3を利用してPull Requestの情報を取得すると、次のようなJSONが取得できる。

- Get a pull request | Pulls GitHub Docs
    - https://docs.github.com/en/rest/reference/pulls#get-a-pull-request

```json
{
  "url": "https://api.github.com/repos/octocat/Hello-World/pulls/1347",
  "id": 1,
  "node_id": "MDExOlB1bGxSZXF1ZXN0MQ==",
  "html_url": "https://github.com/octocat/Hello-World/pull/1347",
  "diff_url": "https://github.com/octocat/Hello-World/pull/1347.diff",
  ....
}
```

ここで、`diff_url`に含まれるURLを取得すると、diffファイルが取得できる。  
たとえば以下のような内容だ。

- https://github.com/budougumi0617/nrseg/pull/14.diff

```diff
diff --git a/inspect.go b/inspect.go
new file mode 100644
index 0000000..9bfc74a
--- /dev/null
+++ b/inspect.go
@@ -0,0 +1,51 @@
+package nrseg
+
+import (
+	"errors"
+	"go/ast"
+	"go/parser"
+	"go/token"
...
```


これを[go-github](https://github.com/google/go-github)で実装すると以下のようになるだろう。
privateリポジトリでも利用できるようにOAuth2で認証を付けておく。

```go
import (
  "github.com/google/go-github/v35/github"
  "golang.org/x/oauth2"
)

// ...

hc := oauth2.NewClient(ctx, oauth2.StaticTokenSource(
  &oauth2.Token{AccessToken: GitHubToken},
))

cli := github.NewClient(hc)
pr, _, _ := cli.PullRequests.Get(ctx, "budougumi0617", "nrseg", 14)
durl := pr.GetDiffURL()
resp, err := hc.Get(pr.GetDiffURL())
```

しかし、OAuth2で認証を付け`Get a pull request`は成功する状態でもdiffファイルの取得で404になる現象が発生した。


# diff_urlはprivateリポジトリでは使えない
調査したところ、`diff_url`に含まれるURLはprivateリポジトリでは利用できないようだった。  
ではどうすると言うかというとメディアタイプを指定することで`Get a pull request`のAPIエンドポイントからdiff情報が取得できた…


実はこのことはAPIドキュメントを読むとちゃんと記載があった。

- Custom media types for pull requests
    - https://docs.github.com/en/rest/reference/pulls#custom-media-types-for-pull-request

> Pass the appropriate [media type](https://docs.github.com/rest/overview/media-types/#commits-commit-comparison-and-pull-requests) to fetch diff and patch formats.

改めてgo-githubで実装すると、diffを取得するコードは次のようになる。

```go
hc = oauth2.NewClient(ctx, oauth2.StaticTokenSource(
    &oauth2.Token{AccessToken: cfg.GitHubToken},
))

cli := github.NewClient(hc)
pr, _, _ := cli.PullRequests.Get(ctx, cfg.Owner, cfg.Repository, cfg.PullRequestNumber)
url := pr.GetURL()

req, _ := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
req.Header.Add("Accept", "application/vnd.github.v3.diff")

resp, err := hc.Do(req)
```


# 終わりに
`diff_url`ってパラメータは何のためにあるんだという気持ちになった。OSSのリポジトリ（publicリポジトリ）だと使えるけれど…  
そして今回もドキュメントはちゃんと読もうね&動作確認ちゃんとしようね案件だった。  
JSONの構造だけみて油断してはいけない。また、publicリポジトリでテストしていたので認証周りのトラップに気づけなかった。使いたい環境でしっかりと動作確認すべきだった。

# 参考
- [Get a pull request | Pulls GitHub Docs](https://docs.github.com/en/rest/reference/pulls#get-a-pull-request)
- [Using curl, how do I download a .diff file from a private repository on github](https://stackoverflow.com/questions/35471316/using-curl-how-do-i-download-a-diff-file-from-a-private-repository-on-github)
- https://github.com/google/go-github
