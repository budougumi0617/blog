+++
title= "Dockerでnode-SASSを使うとNode Sass could not find a binding for your current environment: Linux 64-bit with Node.js X.X..."
date= 2018-04-04T13:20:32+09:00
draft = false
slug = ""
categories = ["docker","webpack"]
tags = ["node", "docker","webpack", "sass"]
author = "budougumi0617"
+++

2,3日ハマっていたのでメモ。  
Dockerでnode-SASSを使ったら以下のエラーが出て動かなかった。

```
sample-app    |         Module build failed: Error: Missing binding /usr/src/app/node_modules/node-sass/vendor/linux-x64-57/binding.node
sample-app    |         Node Sass could not find a binding for your current environment: Linux 64-bit with Node.js 8.x
sample-app    |
sample-app    |         Found bindings for the following environments:
sample-app    |           - OS X 64-bit with Node.js 8.x
...
```

# TL;DR
- ホストの`node_modules`ディレクトリをコンテナにマウントしてしまうとOS差異があったときに`node-sass`の依存性解決に失敗する
- `node_modules`ディレクトリはマウントしないようにしておく
  - `docker run`するときは`-v /your-project-root-in-container/node_modules`
  - `docker compose`するときは`volumes`オプションで`- /your-project-root-in-container/node_modules`

# 問題
Reactとwebpack-dev-serverを使った開発用コンテナを作っていたが、コンテナを起動するとnode-SASSが以下のエラーで動かなかった。  
編集したコードをホットリロードをしながら開発したかったので、当然ホストのReactプロジェクトディレクトリはコンテナにマウントして起動していた。


```
...
sample-app    |         Entrypoint undefined = extract-text-webpack-plugin-output-filename
sample-app    |         [./node_modules/css-loader/index.js??ref--4-1!./node_modules/sass-loader/lib/loader.js??ref--4-2!./src/css/index.scss] ./node_modules/css-loader??ref--4-1!./node_modules/sass-loader/lib/loader.js??ref--4-2!./src/css/index.scss 1.03 KiB {0} [built] [failed] [1 error]
sample-app    |
sample-app    |         ERROR in ./node_modules/css-loader??ref--4-1!./node_modules/sass-loader/lib/loader.js??ref--4-2!./src/css/index.scss
sample-app    |         Module build failed: Error: Missing binding /usr/src/app/node_modules/node-sass/vendor/linux-x64-57/binding.node
sample-app    |         Node Sass could not find a binding for your current environment: Linux 64-bit with Node.js 8.x
sample-app    |
sample-app    |         Found bindings for the following environments:
sample-app    |           - OS X 64-bit with Node.js 8.x
...
```

`Linux`コンテナなのにOSX用のバイナリしかなくて起動に失敗しているらしい。

以下のstack overflowなどをみて`curl`コマンドでLinux用のバイナリを直接配置しておくとか、`npm rebuild node-sass`でリビルドしてしまうなどを試したのだがどれもうまくいかなかった。

**Yarn not rebuilding node-sass**  
https://github.com/yarnpkg/yarn/issues/4867

**Issue to node-sass and Docker**  
https://stackoverflow.com/questions/41942769/issue-to-node-sass-and-docker

**Node/Docker is node-sass not finding installed bindings (via Webpack)**  
https://stackoverflow.com/questions/49138931/node-docker-is-node-sass-not-finding-installed-bindings-via-webpack

# 解決策
結局正解だったのはこれ。

**Docker ALPINE Linux throws node-sass missing binding error**  
https://github.com/sass/node-sass/issues/2165#issuecomment-347043659

プロジェクトディレクトリをまるごとマウントしていると、`node_volumes`ディレクトリまでマウントしてしまう。
そのため、`node_volumes`だけはホストのディレクトリをマウントしないように指定しておく必要があった。
その設定をしておけばわざわざ`npm rebuild`などしなくてもちゃんとLinux向けのバイナリだけが取得された状態で起動出来るようになった。

`docker run`で実行するときは以下のような指定になる。

```
$ docker run -it -v /proj-root-in-host:/proj-root-in-container -v /proj-root-in-container/node_modules -p 8080:8080 --rm sample-app
```

`docker-compose.yml`に設定を書いておくときは以下のような指定になる。



```
  sample-app:
    container_name: sample-app
    build:
      # current directory in build
      context: /proj-root-in-host
      dockerfile: ./Dockerfile
    volumes:
      - /proj-root-in-host:/proj-root-in-container
      - /proj-root-in-container/node_modules
```

改めてdockerコマンドのオプションやdocker-compose.ymlの書き方を調べる良い機会にもなった。

# 参考
**Compose file version 3 reference|volumes**  
https://docs.docker.com/compose/compose-file/#volumes




