+++
title= "[Go] MySQLからScanしたときunsupported Scan, storing driver.Value type []uint8 into type *time.Timeと出てパースに失敗する"
date= 2019-03-31T08:35:30+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go", "sql"]
tags = ["golang", "mysql", "sqlx"]
keywords = ["Go", "MySQL"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

GoでDBを操作するコードを書いている。`jmoiron/sqlx`パッケージを使ってMySQLサーバ内のDBをを操作しようとして表題のエラーに遭遇した。  
MySQLサーバとの接続パラメータを変えることでうまくいったので解決方法をメモしておく。

<!--more-->

# TL;DR
- `jmoiron/sqlx`パッケージを使ってスキーマに`datetime`型を持つレコードを`time.Time`型を持つ構造体に読み込もうとして失敗した
  - `unsupported Scan, storing driver.Value type []uint8 into type *time.Time`
- （`sql.Open`相当の）[`sqlx.Connect`](https://godoc.org/github.com/jmoiron/sqlx#Connect)を使う際にオプションパラメータを指定しておくことで解決することができた
  - `sqlx.Connect("mysql", "root:@(localhost:43306)/sqlx_development?parseTime=true")`

なお、利用したGoとMySQLのバージョンは以下。

- MySQL 5.7
- Go 1.12.1

サンプルコードの`go.mod`は以下の状態。

```
module github.com/budougumi0617/til/go/db/sql-migrate

go 1.12

require (
	github.com/go-sql-driver/mysql v1.4.1
	github.com/jmoiron/sqlx v1.2.0
	google.golang.org/appengine v1.5.0 // indirect
)
```

# 構造体からレコードを書き込めたのに読み込みは失敗した
自分でイチからDB操作を行なうコードを書こうと思い、以下のようなソースコードを作成した。


```go
package main

import (
	"log"
	"time"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

// User is sample structure.
type User struct {
	ID        int       `db:"id"`
	Name      string    `db:"name"`
	Email     string    `db:"email"`
	CreatedAt time.Time `db:"created_at"`
	UpdatedAt time.Time `db:"updated_at"`
}

func main() {
	db, err := sqlx.Connect("mysql", "root:@(localhost:43306)/sqlx_development")
	if err != nil {
		log.Fatalln(err)
	}

	id, err := addUser(db, "John Doe", "john@example.com")
	if err != nil {
		log.Fatal(err)
	}

	user := User{}
	userState := `SELECT * FROM user WHERE id = ?`
	err = db.QueryRowx(userState, id).StructScan(&user)
	if err != nil {
		log.Fatal(err)
	}

	log.Printf("Added user was %v\n", user)
}

func addUser(db *sqlx.DB, n, e string) (int64, error) {
	userState := `INSERT INTO user (name, email, created_at, updated_at) VALUES (?, ?, ?, ?)`
	r := db.MustExec(userState, n, e, time.Now(), time.Now())
	return r.LastInsertId()
}
```

なお、`43306`ポートでリッスンしているMySQL内には、コード中で参照しているデータベース(`sqlx_development`)やテーブル(`user`)をすでに作成している。

```sql
mysql> desc user;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | bigint(20) unsigned | NO   | PRI | NULL    | auto_increment |
| name       | varchar(255)        | NO   |     | NULL    |                |
| email      | varchar(255)        | NO   |     | NULL    |                |
| created_at | datetime            | YES  |     | NULL    |                |
| updated_at | datetime            | YES  |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
5 rows in set (0.01 sec)
```

そして前述のコードを実行すると、`addUser`関数によるレコードの書き込みはうまくいった。
しかし、結果を`User`構造体に読み込むための`db.QueryRowx(userState, id).StructScan(&user)`部分の処理は次のような出力を出して失敗してしまった。

```bash
# 読みやすさのために出力結果に改行を入れている
$ go run adduser.go
2019/03/30 19:33:37 Added user was {6 John Doe john@example.com \
    2019-03-30 10:33:38 +0000 UTC 2019-03-30 10:33:38 +0000 UTC}
```

# `parseTime=true`を指定してMySQLと接続する。
調べてみたら`sql.Open`時にパラメータを指定してDB接続をすれば良いとの事だった。

- time.Time support
  - https://github.com/go-sql-driver/mysql#timetime-support

`jmoiron/sqlx`パッケージの`sql.Open`に該当する操作は`sqlx.Connect`なので、修正は以下のようになる。

```go
db, err := sqlx.Connect("mysql", "root:@(localhost:43306)/sqlx_development?parseTime=true")
```

実際に修正したコードで再度実行したところ、今度はレコード情報を読込できた。


```bash
$ go run adduser.go
2019/03/30 19:33:37 Added user was {6 John Doe john@example.com 2019-03-30 10:33:38 +0000 UTC 2019-03-30 10:33:38 +0000 UTC}
```

# 終わりに
業務では既製のライブラリ化を使ってDBに接続・操作している。  
そのため、DB操作に対する理解が浅く、DBを操作するコードをゼロから書いていくつもりで改めてコードを書いている（OSS使っているのでゼロではないけれど）。  
引き続きCircleCIを使ったE2Eテストの方法などを試して、後日公開する。

# 参考
- Add Scan() support for time.Time
  - https://github.com/go-sql-driver/mysql/issues/9#issuecomment-51552649
- time.Time support
  - https://github.com/go-sql-driver/mysql#timetime-support
- github.com/jmoiron/sqlx
  - https://github.com/jmoiron/sqlx/
- go-sql-driver/mysql
  - https://github.com/go-sql-driver/mysql


