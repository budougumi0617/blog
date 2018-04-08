+++
title= "Goのデバッガ(Delve)のいろいろな起動のしかた(引数を渡して起動、起動中のプロセスにアタッチして起動 etc...)"
date= 2018-04-08T09:12:28+09:00
draft = false
slug = ""
categories = ["go","delve"]
tags = ["golang","delve","dlv","debug"]
author = "budougumi0617"
+++

goでデバッグするときはdelveを使うことが多いと思う。
ユースケース別の起動方法をまとめた。

# TL;DR
- 基本的な起動方法と簡単な操作方法(`dlv debug`/`dlv test`
- 環境変数を設定して起動する(Windows以外)(`ENV=... dlv debug`)
- 引数を渡して起動する(`dlv debug --`)
- 特定のテストケースを指定して起動する(`dlv test -- -test.run`)
- 起動済みのプロセスにアタッチして起動する(`dlv attach`)
- リモートから接続して起動する(`dlv exec` && `dlv connect`)


## 前提
まずはvimを起動する。([`vim-go`も`delve`をサポート](https://github.com/fatih/vim-go/blob/master/CHANGELOG.md#117---march-27-2018)するようになりましたね。)
…ではこの記事が閉じられそうなのでコマンドラインからDelveを使うことを前提とする。

## この記事を読んでもわからないこと
この記事では以下の内容については触れない。

- delveのインストールの仕方
- (若干は最初に触れるが、)delve起動後のデバッグ操作のしかた

# 基本的な起動方法(`dlv debug`/`dlv test`)
普通に起動する場合は`main`パッケージがあるディレクトリに移動した後に以下の通り。
テストコードを使ってデバッグをしたいときも同様。

```bash
$ dlv debug
$ dlv test
```

相対パスで`main`パッケージ(or テストコード)fがある場所を指定することもできる。

```bash
$ dlv debug ./ch03/ex02/
$ dlv test ./ch03/ex02/
```

delvが起動したら`help`コマンドを実行すれば使えるコマンドがわかる。  
ブレークポイントを貼った後(`break (alias: b)`)、`continue (alias: c)`コマンドでプロセスを開始する。  
注意点としては、ブレークポイントを貼るときのファイルの指定は`import`に書くときのPATHになる。  
起動時と同じように相対PATHで指定することは出来ない。

```bash
(dlv) b ./ch03/ex02/tempfile.go:17
Command failed: Location "./ch03/ex02/tempfile.go:17" not found
(dlv) b github.com/budougumi0617/gsp/ch03/ex02/tempfile.go:17
Breakpoint 1 set at 0x10cfac5 for main.main() ./ch03/ex02/tempfile.go:17
```

# 環境変数を設定して起動する(Windows以外)(`ENV=... dlv debug`)
これはDelveの機能ではないが、環境変数を設定してデバッグしたいプロセスを起動するときは以下の通り。

```bash
$ ENV=test dlv debug ./ch03/ex02/
```
`$ENV`に`test`を設定した状態でデバッグ対象を起動する。


# 引数を渡して起動する(`dlv debug --`)
起動時に引数を渡すようなGoのプロセスを実行するときは起動時に`--`以降に引数を書くとデバッグプロセスの引数になる。

`./hello server --config conf/config.toml`と起動するプロセスのデバッグをしたいとき。
```bash
$ dlv exec ./hello -- server --config conf/config.toml
```

# 特定のテストケースを指定して起動する(`-run`相当)(`dlv test -- -test.run`)
コードを修正して`go test`するとき、テストケースが多い場合などはテストケースを指定して実行すると思う。

例：`go test ./ch03/ex02 -run TestCreateTempFile`相当の指定をしてデバッグを開始したい場合。

```bash
$ cd ./ch03/ex02
$ dlv test --  -test.run TestCreateTempFile
```

はい。正直パッケージ指定と`-run`オプションを両立して起動する方法はわからなかった。。。

# 起動済みのプロセスにアタッチして起動する(`dlv attach`)
ローカルで起動済のプロセスに接続する場合は以下のように行う。

例：起動中の`backend`に接続する場合。

```bash
$ ps aux | grep backend
budougumi0617    18370   0.0  0.0  4286184   1040 s003  S+    6:25PM   0:00.00 grep backend
budougumi0617    18363   0.0  0.0 558459612   6916 s002  S+    6:25PM   0:00.02 ./backend
$ dlv attach 18363
Type 'help' for list of commands.
(dlv)
```

`ps`コマンドでプロセス番号を調べてアタッチすれば良い。  
デバッグ終了時、デフォルトではアタッチ先のプロセスも終了してしまうので、それを意図しない場合はエンターを連打しないこと。

```bash
(dlv) q
Would you like to kill the process? [Y/n]
```

# リモートから接続して起動する(`dlv exec` && `dlv connect`)
リモートからデバッガを接続することもできる。  
ただし、サーバ自体のポートを開放する必要があったり、デバッグ元のプロセスをdlv経由で起動しておかないといけないので、あまり使うことはないかな？

`backend`バイナリにリモートデバッグしたい場合。

事前にデバッグ対象を`dlv`コマンドを使って起動しておく。このとき接続を許可するポートなどを指定しておく。
```bash
$ dlv exec backend --headless -l localhost:12345
API server listening at: 127.0.0.1:12345
```

上記のように起動させたら別からデバッグ接続することができる。
```bash
$ dlv connect localhost:12345
```

# 最後に

Macユーザーでこの記事を見て久しぶりにDelve起動したら動かなかった方は以下などを参考にする。  
自分は最近やっとHigh Sierraにアップグレードしたせいで、ハマってしまった。

[could not launch process: EOF](https://github.com/derekparker/delve/issues/1165)


