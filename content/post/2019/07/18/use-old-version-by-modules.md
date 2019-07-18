+++
title= "[Go moudles] module名がgithub.com/account/repo/vXとなっているリポジトリの古いバージョンを使う #golangjp"
date= 2019-07-18T17:56:51+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "modules"]
keywords = ["Go", "golang", "modules"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Go Modulesの使い方をざっくり調べたのでメモ。

<!--more-->

# TL;DR
- go.mod内の`module`名にバージョン名が入っているリポジトリがあった
    - `module github.com/golang-migrate/migrate/v4`
- 古いバージョンは使うときはmodファイル内で`replace`を使えばよい
    - `replace github.com/golang-migrate/migrate/v3 latest => github.com/golang-migrate/migrate v3.5.4`
- ブランチ名が`v1`のようなバージョン名だとブランチではなく、自動的に関連するリリースタグ状態を確認しにいく模様

以下のコードは同じ`github.com/golang-migrate/migrate`リポジトリの各バージョンを取り込む`go.mod`ファイルだ。
バージョンの指定を"イイ感じ"に判断してもらうため、ひとまずコメントのように`latest`をした。
`go mod tidy`を実行後は依存性を正しく解決した状態になっている。

```c
module github.com/budougumi0617/til/go/modules/branch

go 1.12

// go mod tidyコマンドを行う前の手動で書いたgo.modファイルの依存関係
//
// require (
// 	github.com/golang-migrate/migrate v1
// 	github.com/golang-migrate/migrate/v3 latest
//
// 	github.com/golang-migrate/migrate/v4 latest
// )
//
// replace github.com/golang-migrate/migrate/v3 latest => github.com/golang-migrate/migrate v3.5.4

require (
	github.com/golang-migrate/migrate v1.3.2
	github.com/golang-migrate/migrate/v3 v3.5.4

	github.com/golang-migrate/migrate/v4 v4.4.0
	gopkg.in/mattes/migrate.v1 v1.3.2 // indirect
)

replace github.com/golang-migrate/migrate/v3 v3.5.4 => github.com/golang-migrate/migrate v3.5.4+incompatible

```

なお、この記事の内容は`github.com/golang-migrate/migrate`の最新リリースが`v4.4.0`の時に検証された。
また、goのバージョンは1.12.6である。

# go.mod内のmodule名にバージョン名が入っているリポジトリがあった
`golang-migrate/migrate`リポジトリの`go.mod`ファイルを見ると、`module`名が`github.com/golang-migrate/migrate/v4`になっていた。
このような状態のリポジトリで`v3.X.X`系を使いたい場合はどうすればいいのだろうか？

- https://github.com/golang-migrate/migrate/blob/v4.4.0/go.mod

```c
// https://github.com/golang-migrate/migrate/blob/v4.4.0/go.mod
module github.com/golang-migrate/migrate/v4

require (
	cloud.google.com/go v0.37.4
	github.com/aws/aws-sdk-go v1.17.7
	github.com/bitly/go-hostpool v0.0.0-20171023180738-a3a6125de932 // indirect
	github.com/bmizerany/assert v0.0.0-20160611221934-b7ed37b82869 // indirect
	github.com/cockroachdb/apd v1.1.0 // indirect
    // ....
```


# 古いバージョンは使うときはmodファイル内で`replace`を使えばよい
`replace`を使うことで、任意の古いリリース状態を使うことができた。
最初にリポジトリのタグやブランチは以下の状態になっている。

```bash
$ git branch -r
  origin/HEAD -> origin/master
  origin/cli-updates
  origin/master
  origin/v1
  origin/v1-gopkg
```

リリースタグは以下のような状態になっている（一部省略）。
```bash
$ git tag -l
v1.2.0
v1.3.0
v1.3.1
v1.3.2
v3.0.0
v3.0.0-prev0
v3.0.0-prev1
v3.0.0-prev2
...
v3.5.4
v4.0.0
v4.0.1
...
v4.4.0
v4.5.0
```

このような状態のリポジトリのとき、`v3`系を利用して開発する場合は`replace`を使えば良さそうだった。
以下のように手で編集したgo.modファイルを用意する。

```c
module github.com/budougumi0617/til/go/modules/branch

go 1.12

require (
 	github.com/golang-migrate/migrate/v3 latest
)

replace github.com/golang-migrate/migrate/v3 latest => github.com/golang-migrate/migrate v3.5.4
```

該当リポジトリを`import`するファイルを作成しておく。

```go
package main

import (
	"fmt"

	database3 "github.com/golang-migrate/migrate/v3/database"
)

func main() {
	fmt.Println(database3.GenerateAdvisoryLockId("mod test"))
}
```

この状態で`go mod tidy`を実行すれば、正しい`require`と`replace`が指定された状態の`go.mod`ファイルが作成された。

```c
require (
	github.com/golang-migrate/migrate/v3 v3.5.4
)

replace github.com/golang-migrate/migrate/v3 v3.5.4 => github.com/golang-migrate/migrate v3.5.4+incompatible
```


また、`v1`ブランチを使いたいときはどうやって使うのだろう？と以下のように設定を追加してみた。

```c
module github.com/budougumi0617/til/go/modules/branch

go 1.12

require (
	github.com/golang-migrate/migrate v1
	github.com/golang-migrate/migrate/v3 latest
)

replace github.com/golang-migrate/migrate/v3 latest => github.com/golang-migrate/migrate v3.5.4
```

結果は`v1`ブランチではなく、自動的にリリースタグのほうの`v1`系が参照された。

```c
require (
	github.com/golang-migrate/migrate v1.3.2
	github.com/golang-migrate/migrate/v3 v3.5.4
)

replace github.com/golang-migrate/migrate/v3 v3.5.4 => github.com/golang-migrate/migrate v3.5.4+incompatible
```

`v4`系も含めて使おうとすると、go.modは次のようになる。

```c
module github.com/budougumi0617/til/go/modules/branch

go 1.12

require (
	github.com/golang-migrate/migrate v1.3.2
	github.com/golang-migrate/migrate/v3 v3.5.4

	github.com/golang-migrate/migrate/v4 v4.4.0
	gopkg.in/mattes/migrate.v1 v1.3.2 // indirect
)

replace github.com/golang-migrate/migrate/v3 v3.5.4 => github.com/golang-migrate/migrate v3.5.4+incompatible
```

コードの中でどうやって呼ぶかというと、以下のようになる。

```go
package main

import (
	"fmt"

	"github.com/golang-migrate/migrate/driver"
	database3 "github.com/golang-migrate/migrate/v3/database"
	database4 "github.com/golang-migrate/migrate/v4/database"
)

func main() {
	fmt.Println(database3.GenerateAdvisoryLockId("mod test"))
	fmt.Println(database4.GenerateAdvisoryLockId("mod test"))
	fmt.Println(driver.GetDriver("mod test"))
}
```

# 終わりに
（だいぶ前に調べていたのだが、）渋川さんのTweetをみて調べた内容をまとめた。
挙動を見た、というレベルなのでロジック的な部分の確認は出来ていない。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">これのgo moduleがわからん。プロジェクトのルートのgo.modのmoduleは確かにv4ってのが末尾に入っているこれで.../v4でimportできるようになる？v3ってのもreadmeにかかれているけど、これはどこにある？<a href="https://t.co/i1QHEPtBn7">https://t.co/i1QHEPtBn7</a></p>&mdash; 渋川よしき (@shibu_jp) <a href="https://twitter.com/shibu_jp/status/1146184060852563969?ref_src=twsrc%5Etfw">July 2, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

go.modファイルはいつも手で直接編集してしまうのだが、もっとシステマティックに編集するのはどうしたらいいんだろう？
