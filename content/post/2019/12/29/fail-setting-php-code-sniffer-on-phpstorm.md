+++
title= "PhpStorm上でPHP_CodeSnifferのCoding Standardが見つからず、静的解析ができない"
date= 2019-12-29T15:28:44+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["php"]
tags = ["php", "phpstorm", "php codesniffer"]
keywords = ["PhpStrom", "PHP", "PHP CodeSniffer"]
twitterImage = "logos/phpstorm.png"
+++

PhpStormで静的解析を利用しようとしてプロジェクトを新しく設定するたびにハマっているのでメモしておく。  
Dockerを利用してPHP_CodeSnifferを使った静的解析をPhpStormで利用するときの設定がうまくいかなった。

<!--more-->


# TL;DR
- Dockerで用意した環境を使ってPhpStormでPHP開発環境を構築していた
- PHP_CodeSnifferの`Coding Standard`設定が読み込めなかった
  - PHP_CodeSniffer自体の設定は完了していた
- `Tools process timeout`設定を30秒にしたところ解決した。

なお、本記事のバージョン情報は以下のとおり。

- macOS Mojave 10.14.6(Macbook Pro2016 13inch)
- PhpStorm 2019.3.1
- PHP 7.4.1
- docker desktop 2.1.6.0
- Composer 1.8.6
- PHP_CodeSniffer 3.5.3


# DockerとPhpStormを使って開発環境を整える。
PHPはマイナーバージョン間でも環境差異が大きいので、プロジェクトごとにDockerfileを用意して開発している。  
Dockerの構成自体は以下のブログなどを参考にしている。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Ftenkoma.hatenablog.com%2Fentry%2F2019%2F03%2F24%2F023944" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

私が使っている`Dockerfile`の記載は以下の通り（この`Dockerfile`のプロジェクトではWebフレームワークは利用していない）。

```dockerfile
FROM php:7.4.1-cli

RUN pecl install xdebug \
 && docker-php-ext-enable xdebug

RUN apt-get update && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends git \
    && apt-get clean && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/*

# Composer
ENV COMPOSER_HOME /tmp
COPY --from=composer:1.8.6 /usr/bin/composer /usr/bin/composer

RUN apt-get update && apt-get -y install \
    gosu zip unzip\
 && rm -rf /var/lib/apt/lists/*
WORKDIR /app
CMD php -S 0.0.0.0:80
```

上記の`Dockerfile`を次の`docker-compose.yml`を使って起動している。

```yaml
version: '3'
services:
  php-cli:
    build: ./docker
    command: php -v
    volumes:
      - .:/app
```

このプロジェクトでは次の`composer.json`を用意して、テストや静的解析用のツール群もインストールしている。

```json
{
    "name": "budougumi0617/my-php-project",
    "require": {
        "php": ">=7.4.1"
    },
    "require-dev": {
        "squizlabs/php_codesniffer": "3.x",
        "phpunit/phpunit": "^7.5",
        "phpstan/phpstan": "^0.12"
    },
    "config": {
        "platform": {
            "php": "7.4.1"
        }
    },
    "autoload": {
        "psr-4": {
            "": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    }
}
```


あとは、`docker-compose run --rm php-cli composer install`というように実行すればDockerで用意したPHP（とcomposer）で各種ツールやプロジェクトコードが実行できる。


# PhpStorm上でDocker上のPHP環境を使って開発をする。

PhpStormでdocker-compose上のPHP環境を使う設定は以下の公式ページの説明に則ればよい。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://pleiades.io/help/phpstorm/configuring-remote-interpreters.html" data-iframely-url="//cdn.iframe.ly/yJkxTKC?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


PhpStorm上でdocker composeを利用したリモートインタプリタの設定と、`composer install`コマンドの実行が終わっていれば、例えば設定の「Languages & Frameworks」→「PHP」→「Test Frameworks」でもPHPUnitが認識されるようになる。
私はPHP_CodeSnifferなどもインストールしていたので、「Quality Tools」でもツールが認識、バージョンを確認できる状態になっていた。

![PHP_CodeSniffer動作状態](/2019/12/29_sniffer.png)

# PhpStorm上でPHP_CodeSnifferのCoding Standardが見つからない
このPHP_CodeSnifferなどを利用して、コーディング中に静的解析の警告を出すには設定の「Editor」→「Inspections」で静的解析の警告ルールを有効にする必要がある。
具体的には、「Editor」→「Inspections」→「Quality Tools」→「PHP_CodeSniffer validation」という設定だ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://pleiades.io/help/phpstorm/using-php-code-sniffer.html" data-iframely-url="//cdn.iframe.ly/Dhii1yk?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

ただ、今回私のPhpStormではPHP_CodeSnifferを正しく認識出来ていなかったらしく、「Coding Standard」が何も見つからず、設定が正しくできない状態になっていた。

![Coding Standardがなにも見つからない](/2019/12/29_inspections.png)

# PHP_CodeSnifferのタイムアウトルールを30秒に延長する
エラーなども出ないのでどこを修正すればいいのか分からなかったが、結論からいうとDocker上のPHP_CodeSnifferからの結果を待つ時間を30秒にすれば無事「Coding Standard」の項目が表示されるようになった。

![PHP_CodeSniffer設定修正後](/2019/12/29_sniffer2.png)

[公式の設定ガイド](https://pleiades.io/help/phpstorm/using-php-code-sniffer.html)にも以下のような言及があった。
デフォルトのタイムアウト設定が5秒だったので、連携ができる前にPHP_CodeSnifferのプロセスが強制終了されていたようだ。

> ツールプロセスのタイムアウトフィールドで、PhpStormがPHP_CodeSnifferからの結果を待つ時間を指定します。その結果、CPUとメモリの過剰な使用を防ぐためにプロセスが強制終了されます。

# 終わりに
最近PHPやPythonを触るようになって、IDEや仮想環境での開発を行うようになった。新しいプロジェクト（リポジトリ）のコードを触るたびに初期設定でコケているので早く慣れていきたい。
（Goを書くときはVimとローカルホスト上のGo（バージョン管理したことがない）で開発している。）

# 参考
- [PhpStormと連携する必要最小限のDocker環境を作る | こもろぐ @tenkoma](https://tenkoma.hatenablog.com/entry/2019/03/24/023944)
- [リモート PHPインタープリターの構成 - 公式ヘルプ | PhpStorm](https://pleiades.io/help/phpstorm/configuring-remote-interpreters.html)
- [PHP_CodeSniffer - 公式ヘルプ | PhpStorm](https://pleiades.io/help/phpstorm/using-php-code-sniffer.html)
