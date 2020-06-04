+++
title= "[発表資料]ゆるふわ分散トレースはじめました #コネヒトマルシェ"
date= 2020-06-04T16:26:45+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["presentation"]
tags = ["destributed_trace","golang","python"]
keywords = ["コネヒトマルシェ", "Go", "Python", "分散トレース", "PHP"]
twitterImage = "twittercard.png"
+++

2020/06/04のコネヒトマルシェオンラインで登壇したので資料中の参考情報をまとめておく。

- https://connehito.connpass.com/event/176890/

<!--more-->

||【BASE社合同勉強会】コネヒトマルシェオンライン「事業を支えるWeb開発」|
|---|---|
|URL|https://connehito.connpass.com/event/176890/|
|会場|オンライン|
|日時|2020/06/04（木）19:30 〜 20:30|

# 登壇資料

<script async class="speakerdeck-embed" data-id="e702103ddeac4a838505f8136c67fcd1" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# 補足情報

## 分散トレースとは
分散トレースはマイクロサービスに関わるパターン。
- Pattern: Distributed tracing
    - https://microservices.io/patterns/observability/distributed-tracing.html 

W3Cによって仕様も策定され始めている。

- W3C Trace context
    - https://www.w3.org/TR/trace-context/ 

## 分散トレースを実現するためには
分散トレースを実現するにはSaaSを利用したり、OSSを組み合わせる、クラウドベンダーのサービスを利用する。
- SaaS
    - https://newrelic.co.jp/
    - https://www.datadoghq.com/ja/
- OSS
    - https://opentelemetry.io/
    - https://opencensus.io/
    - https://www.jaegertracing.io/
    - https://istio.io/
- クラウドベンダー
    - https://aws.amazon.com/jp/xray/
    - https://cloud.google.com/trace


## BASE BANKの分散トレースへのとりくみ
BASE BANKのサービスのアーキテクチャ構成は次の資料にまとめられている。

- GoでのAPI開発現場のアーキテクチャ実装事例

<script async class="speakerdeck-embed" data-id="a99aee6b391a4fd2bfb63c1b4caf5c9d" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

GoやPythonでContextパターンが標準パッケージで提供されている。

- contextパッケージ
    - https://golang.org/pkg/context/
- コンテキスト変数
    - https://docs.python.org/ja/3/library/contextvars.html


# スライド中に紹介した記事
スライド内で紹介した実装方法については一部をブログにまとめている。

- [[Go] context.TODO()を使って漸進的にcontext対応を始める](/2020/02/21/use-context/)
- [[Python] aiohttpでリクエストスコープのコンテキスト情報を扱う](/2020/04/08/python-use-context-var-in-aiohttp/)
- [[Python] aiohttpで独自形式のアクセスログを出力する](/2020/04/11/python-access-log-in-aiohttp/)
