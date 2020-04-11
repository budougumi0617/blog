+++
title= "[Python] aiohttpで独自形式のアクセスログを出力する"
date= 2020-04-11T11:29:01+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["python"]
tags = ["python","aiohttp"]
keywords = ["aiohttp","python","web","context","Observability","log"]
twitterImage = "logos/python.png"
+++

PythonのWebアプリケーションフレームワーク（WAF）である`aiohttp`でカスタムロガーを使ってアクセスログを出力する。

<!--more-->

# TL;DR
- `aiohttp`にはアクセスログを取得する標準の仕組みがある
    - https://docs.aiohttp.org/en/stable/logging.html#access-logs
- 出力する内容を変更するときは`AbstractAccessLogger`を実装した独自の`AccessLogger`を用意する
    - https://docs.aiohttp.org/en/stable/abc.html#aiohttp.abc.aiohttp.abc.AbstractAccessLogger
- `Application.run_app`メソッドの引数として独自の`AccessLogger`を渡してwebサーバを起動する
- `logger`の設定は忘れずに

```python
from aiohttp import web
from aiohttp.abc import AbstractAccessLogger
from aiohttp.web_app import Application


class CustomAccessLogger(AbstractAccessLogger):
    def log(self, request,response,time: float) -> None:
        self.logger.info(f'access_log')

app: Application = web.Application()
web.run_app(app, access_log_class=CustomAccessLogger)
```

## 環境
- Python 3.8.0
- aiohttp 3.6.2

# 独自のアクセスログを出力したい
Pythonのwebサーバでアクセスログを出力し、ログの中へリクエストヘッダーに含まれる`X-Request-ID`を出力しようと考えた。
利用しているWAFの`aiohttp`のドキュメントをみると、アクセスログとカスタマイズの仕組みが標準でサポートされてたため試してみた。

# AbstractAccessLoggerを実装した独自のAccessLoggerを用いる。
`aiohttp`のアクセスログの仕組みは次のページに記載されている。

- Logging — aiohttp 3.6.2 documentation
    - https://docs.aiohttp.org/en/stable/logging.html#access-logs

基本的にはシグネチャを満たした`log`メソッドを実装すれば良い。

```python
from aiohttp.abc import BaseRequest, AbstractAccessLogger
from aiohttp.web_response import StreamResponse


class CustomAccessLogger(AbstractAccessLogger):
    def log(self,
            request: BaseRequest,
            response: StreamResponse,
            time: float) -> None:
        self.logger.info(f'access_log: {request.remote} '
                         f'"{request.method} {request.path} '
                         f'done in {time}s: {response.status}')
```

この自作クラスの型名をwebサーバ起動時に渡せばこれを利用したアクセスログが出力されるようになる。

```python
app: Application = web.Application()
web.run_app(app, access_log_class=CustomAccessLogger)
```

このクラスはサーバ起動時に自動的に初期化されて利用される。
`self.logger`から呼び出される`logger`オブジェクトは次のように決まる、

- `run_app`メソッドの引数に`access_log`として渡した`logging.logger`オブジェクト
- 何も渡されなかった場合は`logging.getLogger('aiohttp.access')`で取得できる`logger`オブジェクト

上記の例では、`access_log`は未指定なので`logging.getLogger('aiohttp.access')`で取得できるオブジェクトになる。

実際にログ出力される形式、ログレベルは通常の`logging`パッケージのエコシステムに則る。
そのため、適切なログレベルや`handler`を設定しないと出力されない。

`33000`ポートで起動するwebサーバに独自アクセスログを設定するコードが以下になる。

```python
import logging
from aiohttp import web
from aiohttp.abc import BaseRequest, AbstractAccessLogger
from aiohttp.web_app import Application
from aiohttp.web_request import Request
from aiohttp.web_response import StreamResponse


class CustomAccessLogger(AbstractAccessLogger):
    def log(self,
            request: BaseRequest,
            response: StreamResponse,
            time: float) -> None:
        self.logger.info(f'access_log: {request.remote} '
                         f'"{request.method} {request.path} '
                         f'done in {time}s: {response.status}')

async def handler(request: Request) -> StreamResponse:
    return web.Response(text='return\n')


access_logger = logging.getLogger('aiohttp.access')
ch = logging.StreamHandler()
access_logger.setLevel(logging.INFO)
access_logger.addHandler(ch)

app: Application = web.Application()
app.router.add_get('/', handler)

web.run_app(app, port=33000, access_log_class=CustomAccessLogger)
```
`'aiohttp.access'`として登録されている`logging.logger`に`handler`としては標準の`StreamHandler`を付与している。
また、上記の例では`info`メソッドを呼んでいるので、`logger`自体のログレベルを`INFO`以上にセットしておかないと`handler`が呼ばれない。

> Access logs are enabled by default. If the debug flag is set, and the default logger 'aiohttp.access' is used, access logs will be output to stderr if no handlers are attached. Furthermore, if the default logger has no log level set, the log level will be set to logging.DEBUG.

[ドキュメント][access_log]には上記のようにデフォルトなら`DEBUG`レベルで出力されると書いてあったが、明示的にログレベルなどを設定しないと出力されなかった。

[access_log]: https://docs.aiohttp.org/en/stable/logging.html#access-logs

上記コードを起動し、別ターミナルから`curl`コマンドでアクセスしたときのログ出力は以下のようになった。

```bash
$ python -u main.py
======== Running on http://0.0.0.0:33000 ========
(Press CTRL+C to quit)
access_log: 127.0.0.1 "GET / done in 0.0007638730000003591s: 200
```

あとは、`handler`を調整して、`JSON`形式の出力にするなり、ファイルに出力すればよい。

# おわりに
Pythonの`logging`パッケージの`logger`、`handler`まわりのエコシステムの挙動を理解せず、ログが出力できなくてだいぶハマった。
結局公式ドキュメントを読むことで解決したので、やはり言語仕様はひととおり読まないといけないなと痛感した。

# 参考
- Logging HOWTO — Python 3.8.2 ドキュメント
    - https://docs.python.org/ja/3/howto/logging.html#loggers
- Logging — aiohttp 3.6.2 documentation
    - https://docs.aiohttp.org/en/stable/logging.html#access-logs
