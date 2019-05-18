+++
title= "[補足資料] Go Conference 2019 Springでdatabase/sql入門を発表した #gocon"
date= 2019-05-19T13:01:52+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go", "sql", "presentation"]
tags = ["golang", "database", "sql"]
keywords = ["Go Conference", "gocon", "SQL", "database"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


Go Conference(GoCon)で「database/sql入門」というタイトルで発表してきた。
この記事は資料中のリンクや、口頭で説明した内容の補足資料となる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://gocon.connpass.com/event/124530/" data-iframely-url="//cdn.iframe.ly/NRErGxT?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


<!--more-->


# 発表資料

<script async class="speakerdeck-embed" data-id="d179faa0687c48b0800e8331117f7a81" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

サンプルコードは以下になる。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/go-sql-sample" data-iframely-url="//cdn.iframe.ly/H835FMp"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

[database/sql入門 | 2019 Spring Sessions](https://gocon.jp/2019-spring-sessions#a7-s-databasesql%E5%85%A5%E9%96%80)

# データベースサーバについて
DockerHubを使えば使い捨てのRDSをすぐ起動することができる。
DockerHubを見れば公式イメージがある。

- mysql | Docker Hub
  - https://hub.docker.com/_/mysql

# マイグレーションツールについて
`sql-migrate`を推した。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/rubenv/sql-migrate" data-iframely-url="//cdn.iframe.ly/jVPTYh9"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# リポジトリパターンについて
設計パターンとして、DBをあつかうときに大抵は選択するであろうリポジトリパターンを紹介した。
pospomeさんがBOOTHで販売している`pospomeのサーバサイドアーキテクチャ`にGoで書かれた「第4章 詳解リポジトリパターン」という章があるのでそちらを読むと良い（その前の章のアーキテクチャ設計に関する部分も大変参考になる）。
<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fwww.pospome.work%2Fentry%2F2018%2F09%2F30%2F213509" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no" allow="autoplay; encrypted-media"></iframe>

# ORMについて
ORMについては`izumin`さんらが比較記事を書かれているのでそちらを読んでるとよい。

- Go の ORM / query builder 消耗日記 | blog.izum.in
  - https://blog.izum.in/orm-and-query-builders-in-golang-1281c7c1e99f

# database/sqlパッケージの使い方について
GoDocをみるのが一番よい。

- https://golang.org/pkg/database/sql/

たとえば、`sql.Open`関数は一度だけ呼び出せばよい、など、GoDocを見ると使用上の注意がしっかり書かれている。

- https://golang.org/pkg/database/sql/#Open

> The returned DB is safe for concurrent use by multiple goroutines and maintains its own pool of idle connections. Thus, the Open function should be called just once. It is rarely necessary to close a DB.

ざっくりサマリを読みたいときはGo Wikiに記事がある。

- https://github.com/golang/go/wiki/SQLInterface



GoDocサーバの裏でDBは動いていないので実行してもエラーになってしまうが、Examplesにサンプルコードが大量にあるのでそちらを参考にできる。

```go
// https://golang.org/pkg/database/sql/#example_DB_QueryContext

package main

import (
	"context"
	"database/sql"
	"fmt"
	"log"
	"strings"
)

var (
	ctx context.Context
	db  *sql.DB
)

func main() {
	age := 27
	rows, err := db.QueryContext(ctx, "SELECT name FROM users WHERE age=?", age)
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()
	names := make([]string, 0)

	for rows.Next() {
		var name string
		if err := rows.Scan(&name); err != nil {
			// Check for a scan error.
			// Query rows will be closed with defer.
			log.Fatal(err)
		}
		names = append(names, name)
	}
	// If the database is being written to ensure to check for Close
	// errors that may be returned from the driver. The query may
	// encounter an auto-commit error and be forced to rollback changes.
	rerr := rows.Close()
	if rerr != nil {
		log.Fatal(err)
	}

	// Rows.Err will report the last error encountered by Rows.Scan.
	if err := rows.Err(); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s are %d years old", strings.Join(names, ", "), age)
}

```

# テストについて

オブジェクトのモックを作るときは`go-mock`などを利用するだろう。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/mock" data-iframely-url="//cdn.iframe.ly/RivSfnz"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`sql.DB`オブジェクトもmockを作ることができて、`sql-mock`というライブラリもある。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/DATA-DOG/go-sqlmock" data-iframely-url="//cdn.iframe.ly/WRO9UYQ"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

また、Dockerを使ってコードの中からSQLサーバを立ち上げたいときは`test-mysqld`ライブラリが使える。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/lestrrat-go/test-mysqld" data-iframely-url="//cdn.iframe.ly/DDIsjdA"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

コードの中からDockerを立ち上げる`dockertest`というライブラリもあるので、Dockerイメージで立ち上げるときはこちらを使う。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/ory/dockertest" data-iframely-url="//cdn.iframe.ly/SP1TEmw"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


もっと深い話は[@codehex](https://twitter.com/codehex)さんや[`__timakin__`](https://twitter.com/__timakin__)さんの記事を読むと良い。
<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fcodehex.hateblo.jp%2Fentry%2F2018%2F07%2F03%2F211839" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no" allow="autoplay; encrypted-media"></iframe>

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://medium.com/@timakin/go-api-testing-173b97fb23ec" data-iframely-url="//cdn.iframe.ly/e7FAWNQ?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# CircleCIについて
省略してしまったが、CircleCIの設定方法の基本は以下になる。

- Language Guide: Go
  - https://circleci.com/docs/2.0/language-go/

CircleCI上でMySQLなどのDBサーバを使う設定をしたいときは、以下のページを参考に`config.yml`を書いていけば良い。

- Database Configuration Examples
  - https://circleci.com/docs/2.0/postgres-config/

# 最後に
SpeakerDeckにアップしている資料は少し古いので直したいのだが、オリジナルデータのKeyNoteファイルが壊れてしまって開けなくなってしまった。
KeyNoteを閉じる→開くとファイルが壊れている、ということが度々あるのだが使い方が悪いのだろうか…


GoConで発表する（くらいのスキルを身につける）のは長年の夢だったので今回登壇できてとても嬉しい。
ただ、今回の内容はビギナー向けの発表だった。
初心者向けがよくないわけではないが、他の発表者のみなさんのような中級者・上級者向けの発表もできる、あるいは議論できるくらいのスキルを身につけていきたい。


<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">Gopherくんもらう（GoCon登壇して貰えるくらいの発表をできるようになる）のずっと夢だったから嬉しい <a href="https://t.co/DSkcicED1J">pic.twitter.com/DSkcicED1J</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1129926255153799173?ref_src=twsrc%5Etfw">May 19, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Gopherくんかわいい。

# 参考
- https://gocon.jp
- [database/sql入門 | Speakerdeck](https://speakerdeck.com/budougumi0617/introduction-database-sql)
- https://github.com/budougumi0617/go-sql-sample
- [mysql | Docker Hub](https://hub.docker.com/_/mysql)
- https://github.com/rubenv/sql-migrate
- [Go の ORM / query builder 消耗日記 | blog.izum.in](https://blog.izum.in/orm-and-query-builders-in-golang-1281c7c1e99f)
- [【10/5 更新】技術書典5にサークルとして参加します。 | pospomeのプログラミング日記](https://www.pospome.work/entry/2018/09/30/213509)
- https://golang.org/pkg/database/sql/
- https://github.com/DATA-DOG/go-sqlmock
- https://github.com/golang/mock
- https://github.com/ory/dockertest/
- https://github.com/lestrrat-go/test-mysqld
- [mercari.go #1 で「もう一度テストパターンを整理しよう」というタイトルで登壇しました | アルパカ三銃士](https://codehex.hateblo.jp/entry/2018/07/03/211839)
- [GoのAPIのテストにおける共通処理 | Medium](https://medium.com/@timakin/go-api-testing-173b97fb23ec)
- [Language Guide: Go](https://circleci.com/docs/2.0/language-go/)
- [Database Configuration Examples](https://circleci.com/docs/2.0/postgres-config/)
