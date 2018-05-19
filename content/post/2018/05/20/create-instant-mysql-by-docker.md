+++
title= "Dockerで使い捨てのMySQL環境を用意する。事前データを投入して起動する。"
date= 2018-05-20T12:47:20+09:00
draft = false
slug = ""
categories = ["sql","docker"]
tags = ["mysql","docker","sql"]
author = "budougumi0617"
+++

データベースの勉強をしている。  
クエリを書く練習をしたりするにはやはり実際に動いている環境がほしい。  
ただ、ローカルの開発用のデータベースとは別でやりたいとか、使い捨ての環境が欲しくなるときがある。  
そんなときはDockerを利用することで使い捨ての環境を用意できる。

# TL;DR
- MySQLコンテナーをポート割り当てして起動する
- 初期化データを投入したいときは`/docker-entrypoint-initdb.d`にデータをマウントしておく。

なお、ここではDocker自体の説明はしない。
また、初期化データの投入だけ知りたい場合は以下のページが詳しいのでそちらを見ることをおすすめする。

**事前にデータ投入をした MySQL Docker イメージを作る場合は /docker-entrypoint-initdb.d を活用すると便利**  
https://kakakakakku.hatenablog.com/entry/2017/11/08/193031

# Dockerのインストール
Docker自体は以下のページから行う。

**About Docker CE**  
https://docs.docker.com/install/

# MySQLコンテナーの起動
今回はMySQLを使ってSQL環境を作成する。
MySQLは公式コンテナーが以下に用意されている。

https://hub.docker.com/_/mysql/

Docker自体のインストールが完了したら`MySQL`のコンテナーを起動する。  
以下のコマンドを実行すれば自動的にMySQL公式のコンテナーのダウンロードが開始され、内部で`MySQL`が稼働しているコンテナーが開始される。

```bash
$ docker container run --rm -d -e MYSQL_ROOT_PASSWORD=mysql -p 43306:3306 --name mysql mysql:5.7
```

大雑把にいうと、ホストマシンの43336ポートからアクセス出来る使い捨てのMySQLを`mysql`という名前でデーモン起動している。

|オプション名|意味|
|---|---|
|`--rm`|コンテナー終了時に利用したデータを削除する|
|`-d`|バックプロセスとして起動する|
|`-e`|コンテナーに環境変数(`MYSQL_ROOT_PASSWORD`)を与える|
|`-p`|ホストのポートにコンテナーのポートを割り当てる|
|`--name`|実行するコンテナーに`mysql`という名前をつける|
|`mysql:5.7`|`mysql`コンテナーの`5.7`タグを使う|

詳細は以下を確認すればわかる。

**docker container**  
https://docs.docker.com/engine/reference/commandline/container/

コマンド実行後、`docker container ls`(`docker ps`)コマンドで`mysql`コンテナーが起動しているか確認する。

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED                  STATUS              PORTS                     NAMES
4838ea05a813        mysql:5.7           "docker-entrypoint.s…"   Less than a second ago   Up 4 seconds        0.0.0.0:43306->3306/tcp   mysql
```

これで使い捨てのMySQLが起動できた。

# MySQLへのアクセス
上述したとおり、MySQLのコンテナーのポート(3306)をホストマシンのポート(43336)に割り当てているので、ホストマシン（自分のPC）からDBに接続できる。
上記のコマンドの場合は接続するポートは`43336`、`root`のパスワードは`MYSQL_ROOT_PASSWORD`で指定した`mysql`になっている。

```bash
$ mysql -h 127.0.0.1 --port 43306 -uroot -pmysql
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.21 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

# 使い捨てのMySQLコンテナーを終了したいとき
利用を終了するときは以下のコマンドで完了する。

```bash
$ docker container stop mysql
```

前述したとおりの`docker container run`コマンドでコンテナーを起動していた場合は、
`--rm`オプションをつけて起動しているので、利用したコンテナは廃棄される。

# 色々前処理を行った状態でコンテナーを起動したい場合
`CREATE TABLE`や事前データを投入した状態でコンテナーを起動することもできる。
公式のMySQLコンテナーは`/docker-entrypoint-initdb.d`に`sql`ファイルや`gz`ファイルを配置することでコンテナ起動時にMySQLのデータベースを更新することができる。

たとえば、ホストマシン上(Dockerが動いているマシン)に`init`ディレクトリを作り、テーブルを作成する`sql`ファイルとデータを投入する`sql`ファイルを作成しておく。

```SQL
$ cat init/01_initial_db.sql
CREATE DATABASE IF NOT EXISTS todo;
CREATE TABLE IF NOT EXISTS todo.tasks(
  id INT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(100),
  body VARCHAR(1000),
  created_at TIMESTAMP NOT NULL default current_timestamp,
  updated_at TIMESTAMP NOT NULL default current_timestamp on update current_timestamp
);

$ cat init/02_insert_dummy.sql
USE todo;
-- Insert initlal data
START TRANSACTION;
INSERT INTO tasks(title, body) VALUES('First task', 'test data1');
INSERT INTO tasks(title, body) VALUES('Second task', 'test data2');
INSERT INTO tasks(title, body) VALUES('Third task', 'test data3');
INSERT INTO tasks(title, body) VALUES('Dummy Data4', 'long long long long long');
INSERT INTO tasks(title, body) VALUES('Dummy Data5', 'long long long long long');
INSERT INTO tasks(title, body) VALUES('Dummy Data6', 'long long long long long');
INSERT INTO tasks(title, body) VALUES('Dummy Data7', 'long long long long long');
INSERT INTO tasks(title, body) VALUES('Dummy Data8', 'long long long long long');
INSERT INTO tasks(title, body) VALUES('Dummy Data9', 'long long long long long');
INSERT INTO tasks(title, body) VALUES('Dummy Data10', 'long long long long long');
INSERT INTO tasks(title, body) VALUES('Dummy Data11', 'long long long long long');
COMMIT;
```

この`init`ディレクトリを`/docker-entrypoint-initdb.d`として`MySQL`コンテナーにマウントして起動する。
`docker`コマンドで行う場合は`-v`オプションで以下のように行う。このとき`init`ディレクトリのPATHは絶対PATHで指定する必要がある。

```bash
$ docker container run --rm -d -v {$ABSOLUTE_PATH}/init:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=mysql -p 43306:3306 --name mysql mysql:5.7
```

こうして起動した`mysql`コンテナーは常に以下のようにデータが投入されている。
`SELECT`や`JOIN`の練習をするためには、最初にデータを投入しておくと良いかもしれない。

```
mysql -h 127.0.0.1 --port 43306 -uroot -pmysql
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.21 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select *  from todo.tasks;
+----+--------------+--------------------------+---------------------+---------------------+
| id | title        | body                     | created_at          | updated_at          |
+----+--------------+--------------------------+---------------------+---------------------+
|  1 | First task   | test data1               | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
|  2 | Second task  | test data2               | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
|  3 | Third task   | test data3               | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
|  4 | Dummy Data4  | long long long long long | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
|  5 | Dummy Data5  | long long long long long | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
|  6 | Dummy Data6  | long long long long long | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
|  7 | Dummy Data7  | long long long long long | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
|  8 | Dummy Data8  | long long long long long | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
|  9 | Dummy Data9  | long long long long long | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
| 10 | Dummy Data10 | long long long long long | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
| 11 | Dummy Data11 | long long long long long | 2018-05-20 03:02:45 | 2018-05-20 03:02:45 |
+----+--------------+--------------------------+---------------------+---------------------+
11 rows in set (0.00 sec)
```

`/docker-entrypoint-initdb.d`の詳細は以下のリンク先に記載されている。

**事前にデータ投入をした MySQL Docker イメージを作る場合は /docker-entrypoint-initdb.d を活用すると便利**
https://kakakakakku.hatenablog.com/entry/2017/11/08/193031

`Dockerfile`ベースで行う場合は以下のような感じになる。  
https://github.com/budougumi0617/react-golang/tree/master/db
```
FROM mysql:5.7
ENV MYSQL_ALLOW_EMPTY_PASSWORD=yes
ENV MYSQL_ROOT_PASSWORD=""
COPY ./init/01_initial_db.sql /docker-entrypoint-initdb.d/01_initial_db.sql
COPY ./init/02_insert_dummy.sql /docker-entrypoint-initdb.d/02_insert_dummy.sql
COPY my.cnf /etc/mysql/conf.d/my.cnf
```

# 終わりに
データベースなどの環境は、一度壊すとなかなか修復が難しい。  
（そもそも修復が出来るほど簡単or詳しいならば学習など必要ないのかもしれないが）  
カジュアルにいろいろ試したいときはdocker上でやるのがおすすめだ。  

