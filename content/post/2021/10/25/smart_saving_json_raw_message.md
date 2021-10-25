+++
title= "[Go] JSONを構造体にマッピングしつつ生データを保存するUnmarshalJSONの実装方法"
date= 2021-10-25T08:00:09+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","json","gotips"]
keywords = ["Go","golang","json","RawMessage"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

GoではJSONを扱うときでもしっかり型定義に当てはめて利用するのが一般的だ。  
しかし、外部から受け取ったJSONデータは型に当てはめつつ併せて生データも保存しておきたいときがある。  
`Defind Type`をうまく使うとシンプルな`UnmarshalJSON(data []byte)`メソッドを定義できる。

```go
type Event struct {
    ID   string          `json:"id"`
    Type string          `json:"type"`
    Payload Payload      `json:"pyload"`
    // 構造体にマッピングする前のJSONを保存しておきたい
    Raw  json.RawMessage `json:"-"`
}
```

<!--more-->

# TL;DR
- 外部から受け取るJSONは構造が不意に変わることを想定したいときがある
- `UnmarshalJSON(data []byte)`メソッドを使うと独自のJSONパースができる
- 構造体にマッピングすると、構造体に定義されていないデータが欠落する
- JSONの生データを扱うときは`json.RawMessage`を使うとよい
- `Defind Type`で名前を変えると独自定義した`UnmarshalJSON(data []byte)`メソッドを呼ばなくなる

https://play.golang.org/p/n7qvO3vTiV0
```go
type Event struct {
    ID   string          `json:"id"`
    Type string          `json:"type"`
    Payload Payload      `json:"pyload"`
    Raw  json.RawMessage `json:"-"`
}

func (e *Event) UnmarshalJSON(data []byte) error {
  type event Event
  var ee event
  err := json.Unmarshal(data, &ee)
  if err != nil {
    return err
  }
  *e = (Event)(ee)
  e.Raw = data
  return nil
}
```

# GoでJSONを扱うとき
GoではJSONを使うときもしっかり型を意識して取り扱う。
受け取ったJSONデータは`encoding/json`パッケージを使ってデコードして構造体にマッピングする。

https://pkg.go.dev/encoding/json


## `UnmarshalJSON(data []byte)`メソッドを使うと独自のJSONパースができる
`encoding/json`パッケージは通常構造体のフィールドに記載された`json`タグを見て構造体にJSONのデータを当てはめていく。
しかし構造体に対して`UnmarshalJSON(data []byte)`メソッドを定義しておくと、独自ロジックのJSONデコードが行える。

- JSONの変換をカスタマイズするメソッドを生成する | DeNA Codelab
    - https://dena.github.io/codelabs/encodingjson-generator/#0
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://engineering.linecorp.com/ja/blog/flex-message-line-bot-sdk-go/" data-iframely-url="//cdn.iframe.ly/1qPQa7b?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## JSONの生データを扱うときは`json.RawMessage`を使うとよい
また、JSONの元データを扱うときは`json.RawMessage`型を利用する。

- https://pkg.go.dev/encoding/json#RawMessage

# 外部から受け取るJSONは構造が不意に変わることを想定したいときがある
上記の仕組みを使ってJSONデータを使うとき、問題になることがある。
当然といえば当然なのだが、構造体に未定義のキーはJSONデータから構造体へマッピングするときに欠落する。
そのため、たとえば次のようなステップを踏んでいると、HTTPレスポンス中の未定義のキーのデータは喪失する。

- HTTPレスポンスをJSONで受け取る
- 構造体にデコードする
- DBに保存する

しっかりとした仕様の元でJSONを扱っていれば未定義のキーの心配などしなくてもよいのだが、お行儀の悪いAPIだと未定義のデータも混じる可能性が発生しうる。
そのため構造体にマッピングしつつJSON生データの保存するロジックを書きたい時がある。

## 生データを保存する独自`UnmarshalJSON(data []byte)`メソッド
実装の完成形を先に書くと、このような`UnmarshalJSON(data []byte)`メソッドを書けばよい。
```go
type Event struct {
    // もろもろのJSON構造の定義

    // 構造体にマッピングする前のJSONを保存する場所
    Raw  json.RawMessage `json:"-"`
}

func (e *Event) UnmarshalJSON(data []byte) error {
  type event Event
  var ee event
  err := json.Unmarshal(data, &ee)
  if err != nil {
    return err
  }
  *e = (Event)(ee)
  e.Raw = data
  return nil
}
```
やっていることは単純でアンマーシャルしたあと元データを`json.RawMessage`フィールドで参照しているだけだ。
ただ型を定義しなおしてからアンマーシャルしているのがポイントとなる。

```go
type event Event
var ee event
err := json.Unmarshal(data, &ee)
```

## `Defind Type`で名前を変えると独自定義した`UnmarshalJSON(data []byte)`メソッドを呼ばなくなる
型を定義しなおす理由は2つある。

最初の理由は「型を定義しなおさないといけない理由」で、同じ型のまま`json.Unmarshal`関数を使うと`(e *Event) UnmarshalJSON(data []byte)`が再帰的に呼び出されるだけになってしまうからだ。
別定義した`event`型には`(e *event) UnmarshalJSON(data []byte)`メソッドは定義されていない。そのため、通常のアンマーシャルが行われるだけになる。

もうひとつの理由は「型を定義しなおしたほうがよい理由」で、このように定義された`event`型は`Event`型とまったく同じ構造体定義をもつ。
そのため、`(Event)(ee)`のようなキャストも必ず成功する。また、`Event`型の構造体定義に変更が入ったときは`event`型の定義も当然更新される。
メンテフリーで`Event`型の内容に`event`型が追従するため、「いつの間にか壊れていた」ということも発生しない。


# 終わりに
今回のやりかたは`stripe-go`を参考にしている。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/stripe/stripe-go/blob/f1847e5d06c4d13389d10629c6827dc375bfd015/event.go" data-iframely-url="//cdn.iframe.ly/tHrkXv6?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

もっというと[@vvakame](https://twitter.com/vvakame)さんのツイートで見かけたから把握していたという背景もある。
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">おぉー Stripeのライブラリで面白いテクニック見つけた… type hoge Hoge; var v hoge; json.Unmarshal(b, &amp;v) とかしてUnmarshalJSONを適用しない的なやつ</p>&mdash; わかめ@毎日猫がいる (@vvakame) <a href="https://twitter.com/vvakame/status/1307658461711392768?ref_src=twsrc%5Etfw">September 20, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

OSSのコードを読むのは大事。

# 参考
- https://play.golang.org/p/n7qvO3vTiV0
- https://github.com/stripe/stripe-go/blob/f1847e5d06c4d13389d10629c6827dc375bfd015/event.go#L79-L91
- [JSONの変換をカスタマイズするメソッドを生成する | DeNA Codelab](https://dena.github.io/codelabs/encodingjson-generator/#0)
- [line-bot-sdk-go での Flex Message | LINE Engineering](https://engineering.linecorp.com/ja/blog/flex-message-line-bot-sdk-go/)