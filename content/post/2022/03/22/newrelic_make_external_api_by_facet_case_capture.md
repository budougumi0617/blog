+++
title= "[New Relic] FACET CASE句を使って外部サービスのエンドポイントのレスポンスタイムを集計する"
date= 2022-03-22T09:33:50+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["newrelic"]
tags = ["newrelic","apm","facet","nrql"]
keywords = ["newrelic","facet","case","capture"]
twitterImage =  "logos/newrelic.png"
+++

New Relicを使って外部サービスのエンドポイント別のレスポンスタイムを可視化した。  
IDのようなパスパラメータを含むエンドポイントがあるときは`FACET CASE`句と`capture`関数を使うとよい。

<!--more-->

# TL;DR
- New Relicで外部APIのレスポンスタイムを計測するときは`FROM Span WHERE category = 'http'`という条件でNRQLを書く。
- 単純な`FACET`句だと`/users/${USER_ID}`のようなパスパラメータを含むエンドポイントの計測を集約できない。
- パスパラメータを含むエンドポイントは`FACET CASE`句を使って結果をまとめる
- その他のエンドポイントはcapture関数を使って結果を分類する。

次のサンプルNRQLは`https://example.com/external_api`という外部サービスのエンドポイント別レスポンスタイムの中央値を時系列データとして可視化するグラフ。

```sql
SELECT percentile(duration, 50)
FROM Span
WHERE category = 'http' AND appName = 'my-application'
AND http.url LIKE 'https://example.com/external_api%'
FACET http.method, CASES(
    -- https://example.com/external_api/users/${USER_ID}/icon を想定
    WHERE http.url LIKE '%icon' AS '/users/${USER_ID}/icon', 
    -- https://example.com/external_api/users/${USER_ID} を想定
    WHERE http.url LIKE '%users/%' AS '/users/${USER_ID}'
  -- その他のパスパラメータを含まないエンドポイント
) OR capture(http.url, r'https://example.com/external_api(?P<path>.*)')
SINCE 10 day ago TIMESERIES AUTO
```

HTTPメソッドとURLを使ってたとえば次のようなプロットで時系列データをグラフにできる。

1. `PUT /users/${USER_ID}`
2. `GET /users/${USER_ID}`
3. `GET /users/${USER_ID}/icon`
4. `GET /auth`
5. `GET /health`

# New Relic Oneで外部APIのレスポンスタイムを計測したい
New Relic Oneを導入し、分散トレースも確認できる状態のアプリケーションがある。
このアプリケーションが依存している外部APIのレスポンスタイムをエンドポイント別に確認したかった。  
本記事では外部APIは以下のようなエンドポイントを持っている前提で記載する。

- GET https://example.com/external_api/users/${USER_ID}
    - 指定されたIDのユーザー情報を返すエンドポイント
- GET https://example.com/external_api/users/${USER_ID}/icon
    - 指定されたIDのユーザーアイコン画像を返すエンドポイント
- その他パスパラメータを含まないエンドポイント

IDによってレスポンスタイムに大きな違いはないので、「指定されたIDのユーザー情報を返すエンドポイント」としてのレスポンスタイムの集計データを確認したかった。

# `Metric`ではなく`Span`を使う
New Relic One上で書くサービスのサマリーページを見ると「Web transactions time」という`Metric`を使ったグラフがあり「Web external」の情報も確認できる。


![Web transactions time](/2022/03/22_web_transaction_time.png)  
[（参考画像はNew Relic Explorers HubのTopicから拝借）](https://discuss.newrelic.com/t/menu-shortcut-to-add-to-dashboard-for-web-transactions-time-chart/128486)

ここを見るとこのグラフの元データとなる`Metric`からやりたいことができそうだが、`Metric`はプロパティがわかりづらく、NRQLが試行錯誤できなかった。

そこで今回はトランザクショントレースの`Span`からグラフを作成する。
とうぜんアプリケーションでAPMが取得できていて外部Webサービスへのリクエストのメトリクスも取得できている必要がある。

## Goアプリケーションで外部Webサービスへのリクエストのメトリクスを取得する方法
Goで該当メトリクスを取得するにはNew Relic Go Agentの`newrelic.NewRoundTripper`を使う。

- https://pkg.go.dev/github.com/newrelic/go-agent/v3/newrelic#NewRoundTripper

```go
import (
  "net/http"
  "time"

  "github.com/newrelic/go-agent/v3/newrelic"
)

type roundTripperFunc func(*http.Request) (*http.Response, error)

func (f roundTripperFunc) RoundTrip(r *http.Request) (*http.Response, error) {
  return f(r)
}

func HttpClient() *http.Client {
  cli := &http.Client{
    Timeout:   5 * time.Second,
  }

  cli.Transport = func(rt http.RoundTripper) http.RoundTripper {
    return roundTripperFunc(func(req *http.Request) (*http.Response, error) {
      nrt := newrelic.NewRoundTripper(rt)
      nreq := newrelic.RequestWithTransactionContext(req, newrelic.FromContext(req.Context()))
      return nrt.RoundTrip(nreq)
    })
  }(cli.Transport)

  return cli
}
```

このように作った`http.Client`から外部サービスへリクエストを送信すると、NRQLで`FROM Span WHERE category = 'http'`という条件で外部サービスとの通信結果のメトリクスが取得できる。

# 単純な`FACET`句だと`/users/${USER_ID}`のようなパスパラメータを含むエンドポイントの計測を集約できない。
このメトリクスを使ってエンドポイント別のグラフを作る。単純に考えるとNRQL上の`GROUP BY`である`FACET`句を使うことになるだろう。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://docs.newrelic.com/jp/docs/query-your-data/nrql-new-relic-query-language/get-started/nrql-syntax-clauses-functions/" data-iframely-url="//iframely.net/XlR7q7v"></a></div></div><script async src="//iframely.net/embed.js" charset="utf-8"></script>

NRQLは次のようになる。

```sql
SELECT percentile(duration, 50)
FROM Span WHERE category = 'http' AND appName = 'my-application'
AND http.url LIKE 'https://example.com/external_api%'
-- HTTPメソッドとURL別にレスポンスタイムを集約する。
FACET http.method, http.url
SINCE 10 day ago TIMESERIES AUTO
```

しかし、これでは集計に失敗する。  
`FACET`句だけだと、次のようにパスパラメータとして`ID`などが含まれるエンドポイントが`ID`別に集計されてしまう。

1. `PUT https://example.com/external_api/users/123`
2. `GET https://example.com/external_api/users/123`
3. `GET https://example.com/external_api/users/567`
4. `GET https://example.com/external_api/users/333/icon`
5. `GET https://example.com/external_api/auth`
6. `GET https://example.com/external_api/health`

これではIDの数だけグラフにプロットされてしまうので意味があるグラフではなくなる。

# パスパラメータを含むエンドポイントは`FACET CASE`句を使って結果をまとめる
このようなときは`FACET CASE`句を使うとパスパラメータを集約できる。

- https://docs.newrelic.com/jp/docs/query-your-data/nrql-new-relic-query-language/get-started/nrql-syntax-clauses-functions/#sel-facet-cases

パスパラメータを含むエンドポイントは`FACET CASE`句の中で`WHERE LIKE`を使って集約する。
順番にパターンマッチングされているはずなので、一度`WHERE`条件にかかったデータは後続の`WHERE`にはマッチしない。
パスパラメータを含むエンドポイントは`FACET CASE`句に`OR`でつなげるとよい。

```sql
SELECT percentile(duration, 50)
FROM Span
WHERE category = 'http' AND appName = 'my-application'
AND http.url LIKE 'https://example.com/external_api%'
FACET http.method, CASES(
    -- https://example.com/external_api/users/${USER_ID}/icon を想定
    WHERE http.url LIKE '%icon' AS '/users/${USER_ID}/icon', 
    -- https://example.com/external_api/users/${USER_ID} を想定
    WHERE http.url LIKE '%users/%' AS '/users/${USER_ID}'
  -- その他のパスパラメータを含まないエンドポイント
) OR http.url
SINCE 10 day ago TIMESERIES AUTO
```

こうすると次のような集約結果になる。

1. `PUT /users/${USER_ID}`
2. `GET /users/${USER_ID}`
3. `GET /users/${USER_ID}/icon`
4. `GET https://example.com/external_api/auth`
5. `GET https://example.com/external_api/health`


# その他のエンドポイントはcapture関数を使って表示をスッキリする。
`capture`関数を使うとグラフ表示時のラベルをスッキリさせることができる。
正規表現はGoなどと同じRe2形式が使える。
```sql
capture(http.url, r'https://example.com/external_api(?P<path>.*)')
```

`capture`関数も使うとNRQLは次のようになる。


```sql
SELECT percentile(duration, 50)
FROM Span
WHERE category = 'http' AND appName = 'my-application'
AND http.url LIKE 'https://example.com/external_api%'
FACET http.method, CASES(
    -- https://example.com/external_api/users/${USER_ID}/icon を想定
    WHERE http.url LIKE '%icon' AS '/users/${USER_ID}/icon', 
    -- https://example.com/external_api/users/${USER_ID} を想定
    WHERE http.url LIKE '%users/%' AS '/users/${USER_ID}'
  -- その他のパスパラメータを含まないエンドポイント
) OR capture(http.url, r'https://example.com/external_api(?P<path>.*)')
SINCE 10 day ago TIMESERIES AUTO
```


こうすると次のようなグラフのプロットが完成する。

1. `PUT /users/${USER_ID}`
2. `GET /users/${USER_ID}`
3. `GET /users/${USER_ID}/icon`
4. `GET /auth`
5. `GET /health`

# 終わりに
New Relic実践入門という書籍も所持しているのだが、今回のやりかたは載っていないような気がしたので記事にした。  
外部サービスのレスポンスタイムのレスポンスタイムを可視化することができたので、「なんとなく遅くなってません？」などと定性的な話ではなく、定量的に外部連携先と調整をしたり、
レスポンスタイムの悪化の原因部分をより詳細に把握できるようになった。

ちなみにAPMを導入しているサービス自体のエンドポイント別のデータを集計したいときはパスパラメータが入っていてもNRQLが提供するプレースホルダーのような機能で簡単に集約できる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://docs.newrelic.com/docs/data-apis/understand-data/metric-data/query-apm-metric-timeslice-data-nrql/" data-iframely-url="//iframely.net/H4sqV0s"></a></div></div><script async src="//iframely.net/embed.js" charset="utf-8"></script>

# 参考
- New Relic Go agent v3
    - https://pkg.go.dev/github.com/newrelic/go-agent/v3/newrelic
- NRQL syntax, clauses, and functions
    - https://docs.newrelic.com/docs/query-your-data/nrql-new-relic-query-language/get-started/nrql-syntax-clauses-functions/
- Query APM metric timeslice data with NRQL
    - https://docs.newrelic.com/docs/data-apis/understand-data/metric-data/query-apm-metric-timeslice-data-nrql/

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B08WWHBF1J&linkId=4eaab438f5ed5fd02ec41ef70a2b4da9"></iframe>