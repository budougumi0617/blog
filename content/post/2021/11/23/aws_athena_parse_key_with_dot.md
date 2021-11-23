+++
title= "AWS Athenaでドットが含まれたJSONキーをパースする"
date= 2021-11-23T00:57:39+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["aws"]
tags = ["aws","athena","sql"]
keywords = ["Athena","ドット","JSON","AWS"]
twitterImage = "logos/aws.png"
+++

ググっても答えがほとんど見つからなかったのでメモしておく。

<!--more-->

# TL;DR
- AWS Athenaでは`json_extract`を使ってJSON文字列を構造化してパースできる
- ネストしたJSONのキーは`json_extract(body, '$.log.request.method')`のようにアクセスする
- `span.id`のようなキーは`json_extract_scalar(log, '$["span.id"]')`と書けばアクセスできる

```sql
SELECT
json_extract(log, '$.http.request.method') as req_method,
json_extract(log, '$.http.request.path') as req_path,
json_extract(log, '$.http.response.status') as rsp_status,
json_extract_scalar(log, '$["trace.id"]') as trace_id,
json_extract_scalar(log, '$["span.id"]') as span_id,
FROM app_logs
LIMIT 10;
```


# AWS Athenaでは`json_extract`を使ってJSON文字列を構造化してパースできる
アプリケーションログのJSONをS3に大量に保存している。
普段はNew Relicでログを見ているのだが、New Relic上には1か月分のログしかないため昔のログデータについてはAthenaを経由してS3上でのログを確認したかった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://dev.classmethod.jp/articles/athena-json/" data-iframely-url="//cdn.iframe.ly/qbfaw5o"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

今回確認したいログは次のようなJSON形式のログだ。

```json
{
  "container_name": "my_app",
  "ecs_cluster": "prd-cluster",
  "ecs_task_arn": "arn:aws:ecs:ap-northeast-1:3...",
  "log": "{\"level\":\"info\",\"http\":{\"request\":{\"hostname\":\"example.com\",\"method\":\"GET\",\"path\":\"/users\"},\"response\":{\"status\":200}},\"trace.id\":\"9a....\",\"span.id\":\"h34....\"}"
}
```

`log`部分の文字列は次のようなJSON文字列になる。
```json

{
  "level": "info",
  "http": {
    "request": {
      "hostname": "example.com",
      "method": "GET",
      "path": "/users"
    },
    "response": {
      "status": 200
    }
  },
  "trace.id": "9a....",
  "span.id": "h34...."
}
```

このJSONの中身を使ってAthenaでクエリを書きたかった。

# ネストしたJSONのキーは`json_extract(body, '$.http.request.method')`のようにアクセスする
まずAthenaでS3上のログを検索するにはテーブル定義が必要だ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://docs.aws.amazon.com/athena/latest/ug/creating-tables.html" data-iframely-url="//cdn.iframe.ly/Vij1ANI"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

今回のアプリケーションログは次のようなテーブルを使って検索する。

## テーブル定義
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS app_logs (
    container_name string,
    ecs_cluster string,
    ecs_task_arn string,
    log string
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
WITH SERDEPROPERTIES (
	'ignore.malformed.json' = 'true'
LOCATION 's3://our-application-logs/ecs/my_app/';
```

`log`キーの中身は動的に変わるしJSON文字列として保存されている。
もっと`log`キー部分を構造化しようとしてもよいのだが、今回は検索クエリのほうでその中身をパースして利用する。
上記のようなテーブル定義では`log`フィールドはただの文字列だ。しかし、AWS Athenaでは文字列内のJSONの構造を利用してクエリを書くができる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://docs.aws.amazon.com/athena/latest/ug/extracting-data-from-JSON.html" data-iframely-url="//cdn.iframe.ly/N0Xy4PR"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`log`フィールドは有効なJSON文字列なのでネストしたその中身も`json_extract`または`json_extract_scalar`でアクセスすることができる。

```sql
SELECT
json_extract(log, '$.http.request.method') as req_method,
json_extract(log, '$.http.request.path') as req_path,
json_extract(log, '$.http.response.status') as rsp_status,
json_extract_scalar(log, '$.trace.id') as trace_id,
FROM app_logs
LIMIT 10;
```

ただし、`span.id`などのキーはドットが入っているため通常のアクセスでは利用できない。
**`$.trace.id`はネストしているJSON構造と判断され、値が取得できない。**


# `span.id`のようなキーは`json_extract_scalar(log, '$["span.id"]')`と書けばアクセスできる
このようなドットを含むJSONキーは`'$["span.id"]'`と書けばAthenaがパースしてくれる。

```sql
SELECT
json_extract_scalar(log, '$["trace.id"]') as trace_id,
json_extract_scalar(log, '$["span.id"]') as span_id,
FROM app_logs
LIMIT 10;
```

上記と違い`[""]`で囲めば「`trace` オブジェクトの `id` キー」ではなく、「`trace.id`キー」と判断してくれる。

# 終わりに
`span.id`などはNew Relic　Oneが自動付与するキー名であり自分たちでキー名を変更できる要素ではなかった。
`span.id`などはSentryでも記録しているIDだったのでどうしても検索キーとして使いたい背景があった。
だいぶ調べるのに時間がかかったが、目的のログ解析はできそうなのでよかった。

## 余談
ちなみに最近知ったのだがAWSリソースのドキュメントは全部無料のKindleになっている。Auroraとかハイライトつけて読めば理解できるかな？（ただしすごいボリューム）
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B08BF4RZV4&linkId=4b441411818d4610260e9481e04a97ce"></iframe>

# 参考
- Extracting Data from JSON
  - https://docs.aws.amazon.com/athena/latest/ug/extracting-data-from-JSON.html
- Best Practices for Reading JSON Data
    - https://docs.aws.amazon.com/athena/latest/ug/parsing-JSON.html
- Aws Athena - is it possible to query a JSON property that has dots in name
    - https://stackoverflow.com/questions/58137161/aws-athena-is-it-possible-to-query-a-json-property-that-has-dots-in-name

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=491031301X&linkId=6f1444c5de095d2d2832a6b258a3022d"></iframe>