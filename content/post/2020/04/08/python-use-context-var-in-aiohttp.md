+++
title= "[Python] aiohttpでリクエストスコープのコンテキスト情報を扱う"
date= 2020-04-08T09:30:01+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["python"]
tags = ["python","aiohttp"]
keywords = ["aiohttp","python","web","context","Observability"]
twitterImage = "logos/python.png"
+++

Python（`aiohttp`）で透過的にリクエストIDを扱いたいと思い、コンテキストについて調べてみた。

<!--more-->



# TL;DR
- Python
    - https://docs.python.org/ja/3/library/contextvars.html
- aiohttpは標準機能で`contextVar`をリクエストスコープで管理してくれる
    - https://docs.aiohttp.org/en/stable/web_advanced.html#contextvars-support
- ユースケース層やドメイン層でリクエストIDなどを扱うときに便利

```python
REQUEST_ID_CONTEXT = ContextVar('VAR', default='default')


@middleware
async def user_id_middleware(request: Request, handler) -> StreamResponse:
    if request.headers is not None and request.headers.get('x-request-id', '') != '':
        REQUEST_ID_CONTEXT.set(request.headers.get('x-request-id'))
    return await handler(request)

async def handler(request: Request) -> StreamResponse:
    request_id: str = REQUEST_ID_CONTEXT.get()
    return web.Response(text='previous request-id:' + request_id + '\n')
```

## 環境
- Python 3.8.0
- aiohttp 3.6.2

# 透過的にリクエストIDを使いたい
Observability（o11y, 可観測性）や分散トレーシングというような考え方がある。  
あるログが、どのHTTPリクエストに基づいて出力されたのか？どの別サービスのログと紐付いているのか？というようなことを知りたくなる。  
そのようなときにパッケージや関数をまたいでリクエストIDのようなHTTPリクエストに埋め込まれた情報を透過的にログなどに利用したい。  
Goでは`context.Context`を使って透過的な情報を扱うのが一般的だ。Pythonでも似たようなことが出来ないかと調べた。

# ContextVar
まず、Pythonの標準ライブラリを調べたところ、`ContextVar`という仕組みが提供されていた。

- contextvars --- コンテキスト変数
    - https://docs.python.org/ja/3/library/contextvars.html

> このモジュールは、コンテキストローカルな状態を管理し、保持し、アクセスするための API を提供します。 ContextVar クラスは コンテキスト変数 を宣言し、取り扱うために使われます。 非同期フレームワークで現時点のコンテキストを管理するのには、 copy_context() 関数と Context クラスを使うべきです。

HTTPリクエストごとに `copy_context()`を使っって`Context`オブジェクトを生成することで、コンテキスト情報が扱えそうだ。

# aiohttpは標準機能だけでcontextの仕組みを利用できる
PythonでWebアプリを開発するのに、シンプルなウェブアプリケーションフレームワークである`aiohttp`を利用している。
`aiohttp`は、3.5バージョンから`ContextVar`を標準機能としてサポートしていた。

- ContextVars support | Web Server Advanced — aiohttp 3.6.2 documentation
    - https://docs.aiohttp.org/en/stable/web_advanced.html#contextvars-support

> But the context modification made by a handler or middleware is invisible to another HTTP request handling call.

引用部分を読むと、リクエストベースでContextの情報は管理されるようだ。
さっそく以下のようなサンプルコードを書いてみる。

```python
from contextvars import ContextVar
from aiohttp import web
from aiohttp.web import middleware
from aiohttp.web_app import Application
from aiohttp.web_request import Request
from aiohttp.web_response import StreamResponse

# デフォルト文字列として、defaultを設定しておく。
REQUEST_ID_CONTEXT = ContextVar('VAR', default='default')


@middleware
async def user_id_middleware(request: Request, handler) -> StreamResponse:
    # Request Headerにx-request-idがあったとき、Contextの情報を更新する
    if request.headers is not None and request.headers.get('x-request-id', '') != '':
        REQUEST_ID_CONTEXT.set(request.headers.get('x-request-id'))
    return await handler(request)


async def handler(request: Request) -> StreamResponse:
    # Context情報をレスポンスに出力する
    request_id: str = REQUEST_ID_CONTEXT.get()
    return web.Response(text='request-id:' + request_id + '\n')

app: Application = web.Application()
app.middlewares.append(user_id_middleware)
app.router.add_get('/', handler)

web.run_app(app, port=33000)
```


## 動作確認
（ドキュメントを信じるとして）リクエストを同時に渡してみるようなことはしていないが、簡単に動作を確認してみる。  
ターミナルでサンプルコードを起動したあと、別のターミナルから`curl`コマンドでリクエストを送ってみる。

```bash
$ python -u main.py
======== Running on http://0.0.0.0:33000 ========
(Press CTRL+C to quit)

```

何もリクエストヘッダーを付けない場合は、デフォルト文字列が表示される。

```bash
$ curl "http://localhost:33000/"
request-id:default
```

リクエストヘッダーを付けて送信すると、middlewareで付与されたContext情報が付与されている。
リクエストスコープな状態になっているので、次のリクエストを送信するとまたデフォルト文字列に戻っている。

```bash
$ curl -H "x-request-id: user02" "http://localhost:33000/"
request-id:user02
$ curl "http://localhost:33000/"
request-id:default
```

自分で`copy_context()`などを使って管理しなくても、リクエストごとにリクエストスコープで`Context`が扱われているのを確認できた。

# どのように利用するのか？
サンプルコードだとあまりメリットを感じられず、「`handler`関数の中で直接`request`変数からヘッダー読み取ればよくない？」と思うだろう。  
しかし、実際にアプリケーションを作る場合は、レイヤー構造をつくり、`handler`関数内でリクエストを分解し、単純なパラメータとしてユースケース層やドメイン層のコードにデータを受け渡していくだろう。  
そのようなとき、その一連のリクエストしか有効でない情報としてContext情報があると、呼び出し先の関数内でもリクエストIDが取り扱いやすい。  
これでエラー通知やログ出力に透過的にリクエストIDを受け渡せることが確認できた。

# 終わりに
「あるひとくくりの処理の間だけで共有する・有効な情報」はC#やJavaでも`Context`という名前の仕組みで扱っていた。  
AWS LambdaなどでもContextが引数として必要になる。  
「context pattern」で検索してみると、State Patternが出てくるが、もう少し意味は拡張されているかなと思う（オブジェクトの状態ではなく、ある処理中での状態を考えたいから）。

- Stateパターン
    - [https://ja.wikipedia.org/wiki/State_パターン](https://ja.wikipedia.org/wiki/State_%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3)

なので、これはこれでContextパターンとして覚えておいてよさそうだ。  
パターンとしてさまざまな言語やプラットフォームで類似の仕組みが用意されていると、新しい言語でも開発がしやすい。パターンランゲージの重要さを感じた。  


# 参考
- [contextvars --- コンテキスト変数](https://docs.python.org/ja/3/library/contextvars.html)
- [ContextVars support | Web Server Advanced — aiohttp 3.6.2 documentation](https://docs.aiohttp.org/en/stable/web_advanced.html#contextvars-support)
- [https://ja.wikipedia.org/wiki/State_パターン](https://ja.wikipedia.org/wiki/State_%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3)
