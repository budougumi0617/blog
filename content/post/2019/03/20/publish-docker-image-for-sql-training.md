+++
title= "クエリの練習用に20以上のテーブルと大量データを事前投入したMySQLイメージをDocker Hubで公開する"
date= 2019-03-20T19:43:17+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["mysql", "docker"]
tags = ["mysql", "docker", "sql"]
keywords = ["MySQL", "Docker", "SQL"]
twitterImage = "logos/mysql.png"
+++


以前にDockerを使って使い捨てのMySQLを起動する方法を紹介した。今回はMySQL公式のプリセットデータを使ったMySQLコンテナを作成する。
また、ローカルから簡単に操作するMakefileも作成した。

- budougumi0617/mysql-sakila
  - https://cloud.docker.com/repository/docker/budougumi0617/mysql-sakila
- https://github.com/budougumi0617/til/blob/master/sql/Makefile

<!--more-->


# TL;DR
- クエリを書く素振り用に複数テーブル・複数レコードを含むMySQLコンテナイメージを作成した
  - https://cloud.docker.com/repository/docker/budougumi0617/mysql-sakila
  - テーブルやレコードなどはMySQL公式のサンプルデータを利用した
      - https://dev.mysql.com/doc/index-other.html
- ローカルから簡単に操作するためにMakefileも作成した。
  - https://github.com/budougumi0617/til/blob/master/sql/Makefile

# ある程度複雑なテーブルデータが事前投入された使い捨てのMySQLが欲しい
ある程度SQLに慣れてきたので本や雑誌にある少し難しいクエリを試したくなった。
今まではMySQLが配布している`world database`というサンプルデータを含んだDockerイメージを使っていた。


<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/datacharmer/test_db" data-iframely-url="//cdn.iframe.ly/wV6NcRI"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fkakakakakku.hatenablog.com%2Fentry%2F2017%2F11%2F08%2F193031" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no" allow="autoplay; encrypted-media"></iframe>

だが、このデータはテーブルが3テーブルしかなく、複雑なクエリを書くことができない。テーブルを連結したりサブクエリなどを使ったクエリを練習をする場合、それなりにリレーションが張られたテーブル群が必要になる。
毎回ローカルで自前コンテナイメージをビルドするのも面倒なので、作成したイメージをDocker Hubに登録しておくことにした。

# サンプルデータを用意する
Dockerに詰めるデータはやはりMySQL公式のサンプルデータにすることにした。
MySQLで公開されているサンプルデータには`world database`以外にもあり、今回はテーブル数が多い`sakila database`を使う。

- Sakila Sample Database
  - https://dev.mysql.com/doc/sakila/en/

`sakira database`は20以上のテーブルを含んでいる。

- Sakila Sample Database/Structure
  - https://dev.mysql.com/doc/sakila/en/sakila-structure.html


# Dockerfileを書く
まず、事前データを投入したDockerイメージを作るためのDockerfileを作成する。
コンテナ内のデータベースにデータを事前投入する方法は[前述したブログ](https://kakakakakku.hatenablog.com/entry/2017/11/08/193031)や[以前に書いた記事](/2018/05/20/create-instant-mysql-by-docker/)の通りで、`/docker-entrypoint-initdb.d`ディレクトリにSQLのファイルを入れておくだけだ。
データを投入するためのSQL文(`sakila-data.sql`、`sakila-schema.sql`)は以下のページにある`sakila-db.zip`から取り出した。

- Other MySQL Documentation
  - https://dev.mysql.com/doc/index-other.html

ただ、`/docker-entrypoint-initdb.d`ディレクトリに複数SQLファイルがあった場合、SQLの実行順序はファイル名によって決まる。
そのため、テーブルを作成するSQL文が先に実行されるようにリネームしてコピーしている。

```dockerfile
FROM mysql:8.0

COPY sakila-db/sakila-schema.sql /docker-entrypoint-initdb.d/00_sakila-schema.sql
COPY sakila-db/sakila-data.sql /docker-entrypoint-initdb.d/01_sakila-data.sql
COPY sakila-db/sakila.mwb /docker-entrypoint-initdb.d/sakila.mwb
```


上記のDockerfileと配置するSQLファイルをGitHubに公開しておく。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/mysql-sakila" data-iframely-url="//cdn.iframe.ly/Jt5MoLf"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

(リポジトリにはMySQLのバージョンを変えた2つのDockerfileが公開されている)

# Docker Hubに登録しておく
上記のGitHubのコードからコンテナイメージを公開する。公開したイメージが以下だ。

- https://cloud.docker.com/repository/docker/budougumi0617/mysql-sakila

GitHubに公開しておけば、Docker Hubはコミットプッシュに連動して自動ビルドなども行なってくれる。  
上記のDockerfileを以下のビルド設定でDocker Hubに公開した。


![build settings](/2019/03/20_hub_setting.png)

これでGitHubの更新に合わせて、常に最新のDockerイメージを自動でビルドしてくれる。

最低限のコマンドオプションで良ければ、以下のコマンドでDocker Hubで公開されたイメージを利用する事ができる。

```bash
$ docker run  -e MYSQL_ALLOW_EMPTY_PASSWORD=yes budougumi0617/mysql-sakila:5.7
```

# コンテナを起動してSQLの練習をするためのMakefileを作成する。
私はすぐコマンドを忘れて毎回ググっていたため、ローカルで実行するときのためにメモ代わりのMakefileも作成しておいた。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/til/blob/master/sql/Makefile" data-iframely-url="//cdn.iframe.ly/koYdP8y"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

```makefile
.PHONY: help start stop mysql exec run

# 引数がないときはusageを表示する
.DEFAULT_GOAL := help

start: ## Start MySQL Conatainer
	docker container run --rm -d -e MYSQL_ALLOW_EMPTY_PASSWORD=yes \
		-p 43306:3306 --name mysql_til budougumi0617/mysql-sakila:5.7

stop: ## Terminate MySQL Container
	docker container stop mysql_til

mysql: ## Connect to MySQL
	mysql -h 127.0.0.1 --port 43306 -uroot -D sakila

CMD=show tables;
exec: ## Execute query on MySQL ex: make exec CMD="show columns from country"
	mysql -h 127.0.0.1 --port 43306 -uroot -D sakila -e "${CMD}"

FILE=''
run: ## Run quey from file on MySQL ex: make run FILE=./count_city.sql
	mysql -h 127.0.0.1 --port 43306 -uroot -D sakila < ${FILE}

# 各コマンドについたコメントを表示する
help: ## Show options
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}'
```


`3306`ポートは開発で利用しているMySQLでふさがっていることが多いので`43306`ポートで接続するようにしてある。  
用意した`make`コマンドの概要を書くと以下になる。

- `make start` dockerでMySQLを起動する
- `make mysql` `mysql`クライアントを接続する
- `make exec` インスタントにワンコマンドだけ実行したいときに使う
  - `make exec CMD="show columns from country"`のように実行する
- `make run` ホストPC上のSQLファイルを実行する
  - `make run FILE=./count_city.sql`のように実行する
- `make stop` コンテナを停止する。

以下のようにコマンドを使うと一連のSQLの練習ができる。

```bash
$ make start
docker container run --rm -d -e MYSQL_ALLOW_EMPTY_PASSWORD=yes \
		-p 43306:3306 --name mysql_til budougumi0617/mysql-sakila:5.7
e5bd2a3b9edd512e95f2f7b65d422129be81a5b61808b32cb0c7268af4505f9c

$ make mysql
mysql -h 127.0.0.1 --port 43306 -uroot -D sakila
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.25 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SELECT MAX(country.country) AS name, COUNT(city.city_id) AS count
    -> FROM country
    ->   INNER JOIN city
    ->     ON country.country_id = city.country_id
    -> GROUP BY city.country_id
    -> ORDER BY count DESC
    -> LIMIT 10;
+--------------------+-------+
| name               | count |
+--------------------+-------+
| India              |    60 |
| China              |    53 |
| United States      |    35 |
| Japan              |    31 |
| Mexico             |    30 |
| Russian Federation |    28 |
| Brazil             |    28 |
| Philippines        |    20 |
| Turkey             |    15 |
| Indonesia          |    14 |
+--------------------+-------+
10 rows in set (0.00 sec)

mysql> SELECT COUNT(*) FROM film_text;
+----------+
| COUNT(*) |
+----------+
|     1000 |
+----------+
1 row in set (0.00 sec)
```


これでクエリを作る練習が捗りそうだ。


# 終わりに
今回はSQLの練習用のDockerコンテナを作成し、Docker Hubにも登録した。Makefileで引数を読み込む方法やDocker Hubの使い方の勉強にもなった。  
SQL関係の積読本が溜まっているので、このコンテナを使って勉強を進めたい。

- https://cloud.docker.com/repository/docker/budougumi0617/mysql-sakila


# 参考
- [Dockerで使い捨てのMySQL環境を用意する。事前データを投入して起動する](/2018/05/20/create-instant-mysql-by-docker/)
- 事前にデータ投入をした MySQL Docker イメージを作る場合は /docker-entrypoint-initdb.d を活用すると便利
  - https://kakakakakku.hatenablog.com/entry/2017/11/08/193031
- Creating Repositories | Docker Hub
  - https://docs.docker.com/docker-hub/repos/#creating-repositories
- Set up Automated builds | Docker Hub
  - https://docs.docker.com/docker-hub/builds/
