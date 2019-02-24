+++
title= "GoでJSONのUnmarshalがシンタックスエラーで失敗した時、エラー周辺の文字列を表示する"
date= 2019-02-24T21:08:31+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["go","json"]
keywords = ["Go", "JSON", "Unmarshal"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++
`json.Unmarshal`に失敗したとき`json.SyntaxError`が取得できた場合は文字列の中のどの位置でパースに失敗したのか知ることができる。
<!--more-->

# TL;DR

- `json.Unmarshal`はUnmarshalに失敗した時`json.SyntaxError`を返す場合がある
- `json.SyntaxError`はインデックス情報(`SyntaxError.Offset`)を持っている
- `SyntaxError.Offset`を使うと通常のエラーより少し解析がラクにできるかもしれない

# json.SyntaxErrorとは

Twitterを見ていたところ、以下のTweetを読んだので実際に試してみた。


<blockquote class="twitter-tweet" data-lang="ja"><p lang="en" dir="ltr">TIL Go json syntax errors give you the offset, so you can provide more context if you want, the single char gets a little confusing <a href="https://t.co/r2dR9JkBjt">pic.twitter.com/r2dR9JkBjt</a></p>&mdash; TJ Holowaychuk 🙃 (@tjholowaychuk) <a href="https://twitter.com/tjholowaychuk/status/1098872828110229505?ref_src=twsrc%5Etfw">2019年2月22日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

`json.SyntaxError`の定義は以下の通りだ。

https://golang.org/pkg/encoding/json/#SyntaxError

```go
type SyntaxError struct {
        Offset int64 // error occurred after reading Offset bytes
        // contains filtered or unexported fields
}
```

オフセット情報を含んだ`error`インターフェースを満たす構造体らしい。

# json.Unmarshalで失敗したときのエラー出力を確認する
まずTweetのような`json.Unmarshal`を失敗する簡単なサンプルコードを用意した。

https://play.golang.org/p/PgLmO9fo0nX

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

func main() {
	bd := []byte(data)
	var v struct {
		Timestamp    string `json:"timestamp"`
		ParentTotals struct {
			C int `json:"C"`
		} `json:"parent_totals"`
	}
	if err := json.Unmarshal(bd, &v); err != nil {
		// 2009/11/10 23:00:00 invalid character '"' after object key:value pair
		log.Fatal(err)
	}
	fmt.Println(v)
}

// "c": 52.57732", の部分に「"」が足りないためinvalidなJSON文字列。
const data = `
{
      "timestamp": "2019-02-24 07:07:03",
      "parent_totals": {
        "C": 0,
        "b": 0,
        "d": 0,
        "f": 9,
        "h": 102,
        "M": 0,
        "c": 52.57732",
        "N": 0,
        "p": 34,
        "m": 58,
        "diff": null,
        "s": 1,
        "n": 194
      }
}`
```

`data`文字列は`52.57732",`部分が不正なのでUnmarshalに失敗する。
出力される`log.Fatal`の結果は以下だ。取得された`error`オブジェクトの`Error()`から取得できる文字列だけだと正直どこが問題だったのかわかりにくい。

```
invalid character '"' after object key:value pair
```

# json.SyntaxErrorを使ってもう少し情報を入手する

`SyntaxError.Offset`と元データの文字列を使うともう少しエラーに関係する部分の情報が取得できる。試し先程のコードのエラーハンドリング部分を`SyntaxError.Offset`から前後15文字の情報も出力するようにしてみる。

https://play.golang.org/p/axUQ1WnMhvV

```go
	if err := json.Unmarshal(bd, &v); err != nil {
		if err, ok := err.(*json.SyntaxError); ok {
			fmt.Println(string(bd[err.Offset-15:err.Offset+15]))
		}
		log.Fatal(err)
	}
```

以下の情報が出力された。これくらいの情報があると、すぐエラーの位置がわかりやすそうだ。

```
 "c": 52.57732",
        "N":
2009/11/10 23:00:00 invalid character '"' after object key:value pair
```

# 終わりに

他にも少しいじってみたところ、構造体定義が問題のときは`json.SyntaxError`は取得できなかった。
あくまでも文字列がJSONとして読めないときにのみしか発生しないようだ。

https://play.golang.org/p/J6P03WyIwhw

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

func main() {
	bd := []byte(data)
	var v struct {
		Timestamp    string `json:"timestamp"`
		ParentTotals struct {
			C int `json:"C"`
		} `json:"parent_totals"`
	}

	if err := json.Unmarshal(bd, &v); err != nil {
		// 真にはならない
		if err, ok := err.(*json.SyntaxError); ok {
			fmt.Println(string(bd[err.Offset-1:]))
		}
		// 2009/11/10 23:00:00 json: cannot unmarshal string into Go struct field .C of type int
		log.Fatal(err)
	}
	fmt.Println(v)
}

// `json:"C"と`json:"c"が宣言されていない構造体でUnmarshalしようとすると失敗する正しいJSONシンタックスの文字列。
const data = `
{
      "timestamp": "2019-02-24 07:07:03",
      "parent_totals": {
        "C": 0,
        "c": "52.57732"
      }
}`
```

# 参考
- https://golang.org/pkg/encoding/json/#SyntaxError
