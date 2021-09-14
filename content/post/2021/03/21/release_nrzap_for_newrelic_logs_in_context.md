+++
title= "[Go] go.uber.org/zapでNew Relic logs in contextをするためのライブラリを書き始めた"
date= 2021-03-21T10:30:00+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go","newrelic"]
tags = ["newrelic","o11y","golang"]
keywords = ["New Relic","Go","logs in context","o11y"]
twitterImage = "logos/newrelic.png"
+++

`uber-go/zap`を使いつつNew Relic Logs in contextを利用するためのライブラリを作り始めた。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/nrzap" data-iframely-url="//cdn.iframe.ly/2Ijbpvo?card=small""></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->


# New Relic logs in context
- [Configure logs in context for Go][logs_in_context]

New Relic Oneを使っていると分散トレーシングを取得することができる。  
そしてNew Relic Oneはログも取得することができるためログに適切な情報を含んでいれば分散トレースとログを紐付けて確認することができる。

GoでNew Relic Oneを利用する場合は `newrelic/go-agent/v3` を使ってもろもろを仕込む。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/newrelic/go-agent" data-iframely-url="//cdn.iframe.ly/MG601Cq?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`newrelic/go-agent/v3` に含まれているlogs in context用の実装は `logrus` logger向けの実装しかなく、私が普段使っている `zap` logger向けの実装が存在しなかった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/uber-go/zap" data-iframely-url="//cdn.iframe.ly/n67ZiPe?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

そこで、せっかくなので自作してみようと考えた。

# logs in contextを実現するために必要なこと
- https://github.com/newrelic/go-agent/tree/master/v3/integrations/logcontext

New Relicである分散トレースにログを結びつけるにはどうしたらいいか？  
既存のプラグインのコードを確認したところ、ログのJSONのルートへ`logscontext` pkgに定義されるキーワードを使って分散トレースIDなどを埋め込めば良さそうだった。  
Webアプリケーションを実装している場合、これらの情報は`context.Context`の中から`newrelic.LinkingMetadata`を取り出せば取得できる。  
`zap`logger自体は`context.Context`を扱うようなインターフェイスではないため、まずは`context.Context`から`[]zap.FIeld`として情報を取り出すヘルパー関数だけ作ることにした。

# nrzap
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/nrzap" data-iframely-url="//cdn.iframe.ly/2Ijbpvo?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

nrzapは`zap`でNew Relic Logs in contextをよしなにするための実装ライブラリ（の予定）だ。

## GtNrMetadataFields
2021/03/21現在nrzapは1つの機能しか提供していない。  
`GtNrMetadataFields` 関数は分散トレースIDなどの情報を`zap`loggerの`Info`メソッドや`Error`メソッドの引数に渡す`[]zap.Field`として生成する。  
単純な利用方法だとこのようになる。


```go
func ExampleHandler(w http.ResponseWriter, r *http.Request) {
	logger, _ := zap.NewProduction()
    defer logger.Sync()
    // nrgorillaなどでcontextから*newrelic.Transactionが取れる前提
    nrfs := nrzap.GetNrMetadataFields(r.Context())
    logger.Info("failed to fetch URL",
        // Structured context as strongly typed Field values.
        zap.String("url", url),
        zap.Int("attempt", 3),
        zap.Duration("backoff", time.Second),
        nrfs...,
    )
}
```

このようにロギングメソッドを呼ぶと、New Relicが識別するためのログトレースIDなどがログの中へ含まれるようになる。

# 注意
なお、今回作成した関数はあくまでログのJSONに情報を埋め込むだけなので、実際に利用するには次の前提条件がある。

- すでに分散トレースをNew Relic One上で計測する仕組みが存在する
- 別途アプリケーションのログをNew Relicに送信している

この条件を満たしている状態でライブラリを使ってログを加工すればlogs in contextが実現する。

# 終わりに
実はlogs in contextはまだあまり使いこなせていないので、もうすこし使い込んだら他の実装も増えるかもしれない。


# 参考
- https://github.com/budougumi0617/nrzap
- https://github.com/uber-go/zap
- https://github.com/newrelic/go-agent/tree/master/v3/integrations/logcontext
- [Configure logs in context for Go][logs_in_context]
- [AWS FireLens plugin for log forwarding][firelens]

[firelens]: https://docs.newrelic.com/docs/logs/enable-log-management-new-relic/enable-log-monitoring-new-relic/aws-firelens-plugin-log-forwarding/


[logs_in_context]: https://docs.newrelic.com/docs/logs/enable-log-management-new-relic/logs-context-go/configure-logs-context-go/