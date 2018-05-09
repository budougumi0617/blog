+++
title= "go getで取得されるコードはmasterブランチ(HEAD)がデフォルトではない #golang"
date= 2018-05-10T09:00:17+09:00
draft = false
slug = ""
categories = ["go"]
tags = ["golang", "goget"]
author = "budougumi0617"
+++

gocon中の@ymotongpooさんのTweetが気になったので調べた。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">前から結構言ってるんだけど、go getで最新取られたくない場合は go1 というタグを切れば良いです <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; Yoshifumi Yamaguchi 🇺🇸 (@ymotongpoo) <a href="https://twitter.com/ymotongpoo/status/985418278288764928?ref_src=twsrc%5Etfw">2018年4月15日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


# TL;DR
- go 1.Xを使っているとき、`go get`で取得されるコードはまず`go1` branch or tagから取得される
- branchにもtagにも`go1`が存在しなかったしなかったときに`master`のコードが取得される

# 実際に仕様を見てみる

golang.orgの`go get`コマンドのページを見ると確かに`go1`について言及されている。

https://golang.org/cmd/go/#hdr-Download_and_install_packages_and_dependencies

> When checking out or updating a package, get looks for a branch or tag that matches the locally installed version of Go. The most important rule is that if the local installation is running version "go1", get searches for a branch or tag named "go1". If no such version exists it retrieves the default branch of the package.

# 実際にコードを見てみる

`go get`の実装コードを見てみる。何やら`tags`というスライスの中から`go1`を探しているコードがある。

https://github.com/golang/go/blob/master/src/cmd/go/internal/get/get.go#L513-L528

```go
// selectTag returns the closest matching tag for a given version.
// Closest means the latest one that is not after the current release.
// Version "goX" (or "goX.Y" or "goX.Y.Z") matches tags of the same form.
// Version "release.rN" matches tags of the form "go.rN" (N being a floating-point number).
// Version "weekly.YYYY-MM-DD" matches tags like "go.weekly.YYYY-MM-DD".
//
// NOTE(rsc): Eventually we will need to decide on some logic here.
// For now, there is only "go1". This matches the docs in go help get.
func selectTag(goVersion string, tags []string) (match string) {
	for _, t := range tags {
		if t == "go1" {
			return "go1"
		}
	}
	return ""
}
```

（コメントの通り、現状go1.X系しか存在しないので、手抜き実装になっている。）

上記引数の`tags`は`vcsCmd`オブジェクトの`tags`メソッドで生成される`string`スライスだ。
https://github.com/golang/go/blob/814c749c8fa815a8ddf8184bcac8990ef0dea006/src/cmd/go/internal/get/get.go#L497-L498

```go
	// Select and sync to appropriate version of the repository.
	tags, err := vcs.tags(root)
```

https://github.com/golang/go/blob/814c749c8fa815a8ddf8184bcac8990ef0dea006/src/cmd/go/internal/get/vcs.go#L472-L486

```
// tags returns the list of available tags for the repo in dir.
func (v *vcsCmd) tags(dir string) ([]string, error) {
	var tags []string
	for _, tc := range v.tagCmd {
		out, err := v.runOutput(dir, tc.cmd)
		if err != nil {
			return nil, err
		}
		re := regexp.MustCompile(`(?m-s)` + tc.pattern)
		for _, m := range re.FindAllStringSubmatch(string(out), -1) {
			tags = append(tags, m[1])
		}
	}
	return tags, nil
}
```

`v.tagCmd`のコマンドによって`tags`の一覧が取得されるが、`git`の場合は`git show-ref`コマンドで、このコマンドを使うと`tag`と`branch`の一覧が取得できる。

https://github.com/golang/go/blob/814c749c8fa815a8ddf8184bcac8990ef0dea006/src/cmd/go/internal/get/vcs.go#L151-L155

```go
	tagCmd: []tagCmd{
		// tags/xxx matches a git tag named xxx
		// origin/xxx matches a git branch named xxx on the default remote repository
		{"show-ref", `(?:tags|origin)/(\S+)$`},
	},
```

ちなみに`git show-ref`コマンドの出力はこんな感じ。

```bash
$ git show-ref
814c749c8fa815a8ddf8184bcac8990ef0dea006 refs/heads/master
814c749c8fa815a8ddf8184bcac8990ef0dea006 refs/remotes/origin/HEAD
...
814c749c8fa815a8ddf8184bcac8990ef0dea006 refs/remotes/origin/master
...
6174b5e21e73714c63061e66efdbe180e1c5491d refs/tags/go1
2fffba7fe19690e038314d17a117d6b87979c89f refs/tags/go1.0.1
cb6c6570b73a1c4d19cad94570ed277f7dae55ac refs/tags/go1.0.2
30be9b4313622c2077539e68826194cb1028c691 refs/tags/go1.0.3
...
```

ここまでの一連の流れで、`tag`、`branch` の一覧の中に`go1`があった場合は、そこから`go get`される実装になっていることがわかった。

https://github.com/golang/go/blob/814c749c8fa815a8ddf8184bcac8990ef0dea006/src/cmd/go/internal/get/get.go#L506

```
vcs.tagSync(root, selectTag(vers, tags))
```

# 終わりに
コマンドの実装もコードで簡単に追えるのが`go`のいいところだなと改めて思った。
`go2`が出たら思い出すと良いかも？
