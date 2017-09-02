+++
title= "docker-compose上のRailsのデバッグを行う"
date= 2017-09-02T12:16:06+09:00
draft = false
slug = ""
categories = ["RubyOnRails", "Docker"]
tags = ["RubyOnRails","Docker", "Debug"]
author = "budougumi0617"
+++

Rails本の写経を`docer-compose`で行なったときのTips。

# TL;DR
`docke-compose`で作ったRubyOnRailsコンテナで`binding.pry`によるデバッグを行えるようにする。

# 前提
`doker-compose`でRails、Spring用のコンテナなど、複数コンテナを起動する形のRails環境を構築した。基本構成は以下の記事に習っている。

[高速に開発できる Docker + Rails開発環境のテンプレートを作った](http://qiita.com/kawasin73/items/2253523be18e5afd994f)

# 事前準備
Railsをデバッグ実行するために必要な設定ファイルの準備をする。

コンテナの標準入出力にアタッチするために、Dockerの設定をしておく。

[docker-compose.yml](https://github.com/budougumi0617/RubyTraining/blob/master/baukis/docker-compose.yml#L18)
```yaml
services:
  rails: &app_base
    tty: true
    stdin_open: true
```

ブレークポイントを貼るための`binding.pry`をするためのGemを追加する。
Gemfileは`pry-rails`の他に`pry-byebug`も追加しておくとステップ実行が出来る。

[Gemfile](https://github.com/budougumi0617/RubyTraining/blob/master/baukis/Gemfile#L38)
```gemfile
group :development, :test do
  gem 'pry-rails'
  gem 'pry-byebug'
end
```

Railsプロジェクトのrootディレクトリに`.pryrc`ファイルを配置しておけばデバッグ中に利用できるエイリアスが貼れる。（他のも色の設定とか出来るらしい）

[.pryrc](https://github.com/budougumi0617/RubyTraining/blob/master/baukis/.pryrc)
```
if defined?(PryByebug)
  Pry.commands.alias_command 'c', 'continue'
  Pry.commands.alias_command 's', 'step'
  Pry.commands.alias_command 'n', 'next'
  Pry.commands.alias_command 'f', 'finish'
end
```

デバッグしたいメソッドに`binding.pry`の記載をして、`docker-compose up`コマンドでRailsを起動して、該当メソッドが実行される操作を行う。
出力がこんな状態になれば`pry`でデバッグが開始できる状態になっている。

```
rails_1   | From: /app/app/controllers/staff/customers_controller.rb @ line 4 Staff::CustomersController#index:
rails_1   |
rails_1   |     2: def index
rails_1   |     3:   binding.pry
rails_1   |  => 4:   @search_form = Staff::CustomerSearchForm.new(params[:search])
rails_1   |     5:   @customers = @search_form.search.page(params[:page])
rails_1   |     6: end
```

# 操作
上記を行うと、`binding.pry`が記載された場所で実行がストップしている状態になっている。ここからRailsが実行されているコンテナにアタッチする。
まずアタッチするRailsコンテナを`docker ps`コマンドで調べる。

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
7fdf3a1fd61c        baukis_rails        "prehook 'ruby -v'..."   5 seconds ago       Up 4 seconds        0.0.0.0:3333->3000/tcp   baukis_rails_1 # attachしたいのはこれ
cb68afd353c8        baukis_spring       "prehook 'ruby -v'..."   5 seconds ago       Up 4 seconds                                 baukis_spring_1
7b86e4f07b20        postgres:9.6.2      "docker-entrypoint..."   6 seconds ago       Up 7 seconds        0.0.0.0:5432->5432/tcp   baukis_db_1
```

その後、`docker attach`コマンドでコンテナに接続する。反応が無いように見えるときはエンターキーを押下すれば`pry`が始まる。

```
$ docker attach baukis_krails_1 # docker psコマンドの結果のNAMEに書いてあった名前
[1] pry(#<Staff::CustomersController>)>
[2] pry(#<Staff::CustomersController>)>
```

# デバッグを終了するとき
pry自体は`quit`コマンドで終了できる。そのあと、
**`Ctrl + C`などでdockerの接続を終了すると、コンテナ自体が終了してしまう。**
デタッチするときは`Ctrl-P Ctrl-Q`で抜ける。


# 参考

[高速に開発できる Docker + Rails開発環境のテンプレートを作った](http://qiita.com/kawasin73/items/2253523be18e5afd994f)

[Dockerコンテナからのデタッチ](http://tech.withsin.net/2015/09/30/docker-container-detach/)

[pry-byebugでrubyをデバッグする](http://qiita.com/AknYk416/items/6f0bec58712edaf4940e)
