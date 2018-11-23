+++
title = "[Go] JSON内の数字や文字列が混じった配列を構造体にUnmarshalする"
date = 2018-11-23T23:22:27+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "json"]
+++

以下のようなJSONデータはGoではパースしずらい。
理由は、配列に複数の型が含まれていてGoの配列としては`[]interface{}`にするしかないことと、名前(`key`)が設定されていないのでマッピングしにくいからだ。
これを無理やりUnmarshalしたときのメモ。

<!--more-->

```json
{
  "totals": [
    1,
    7,
    3,
    "42.85714",
    4
  ]
}
```

# TL;DR
- Goは構造体の`UnmarshalJSON`を独自実装すると、Unmarshal時の挙動も独自定義できる
- JSON配列は`Unmarshal`すると`[]interface{}`となるので一つ一つ型キャストすれば値が取れる

最初にコードだけ書いておくと以下になる。

https://play.golang.org/p/dB01YLzxExL

```go
package main

import (
	"encoding/json"
	"errors"
	"testing"
)

// TotalsArray defined for totals array
type TotalsArray struct {
	Files         int
	CoverageRatio string
}

// UnmarshalJSON 独自定義したUnmarshal方法でUnmarshal時の動きを変える
func (ta *TotalsArray) UnmarshalJSON(data []byte) error {
	// まず引数のdataとして渡ってきたJSONから`[]interface{}`を取得する
	var row []interface{}
	err := json.Unmarshal(data, &row)
	if err != nil {
		return err
	}

	// 決め打ちで型キャストして取得した情報で構造体の初期化を行なっていく。
	f, ok := row[0].(float64)
	if !ok {
		return errors.New("failed type cast f")
	}
	ta.Files = int(f)

	c, ok := row[3].(string)
	if !ok {
		return errors.New("failed type cast c")
	}
	ta.CoverageRatio = c

	// 他のフィールドは省略

	return nil
}

func TestTotalsArray_UnmarshalJSON(t *testing.T) {
	// 複数型が混じった配列を含むJSON配列。Keyがないので個別にUnmarshalできない
	jsondata := `
{
  "totals": [
    1,
    7,
    3,
    "42.85714",
    4
  ]
}`
	// 期待するUnmarshal後の構造体の状態
	want := TotalsArray{
		Files:         1,
		CoverageRatio: "42.85714",
	}

	// "totals" key部分をTotalsArrayとして解釈する構造体
	var got struct {
		Totals TotalsArray `json:"totals"`
	}

	json.Unmarshal([]byte(jsondata), &got) // {Files:1, CoverageRatio:"42.85714"}

	if got.Totals != want {
		t.Fatalf("want:\n%+v\nbut, got:\n%#v", want, got.Totals)
	}
}

```

# JSON配列をうまくUnmarshalできない
Goで3rdパーティのAPIクライアントを書こうとしたら、以下のような`totals`配列を含んだJSONデータを出力されていた。

```json
{
  "totals": [
    1,
    7,
    3,
    "42.85714",
    4
  ]
}
```

GoでJSONデータをUnmarshalするときは通常は構造体のフィールドにタグをつけてマッピングすることでUnmarshalしていく。
```go
type User struct {
	Name      string `json:"name"`
	Email     string `json:"email"`
}
```

が、今回`totals`はオブジェクトではなく配列なので`Key`が設定されていない。また、intと文字列が混じる配列のため、単純な`[]int`でも受けられない。
そのため別手段で強引に構造体に`Unmarshal`することにした。

# 構造体に`UnmarshalJSON([]byte) error`を定義する
まず、最終的に`totals`配列のデータを格納する構造体を定義する（一部省略）。

```go
// TotalsArray defines totals responses.
type TotalsArray struct {
	Files         int
	CoverageRatio string
}
```

（配列に`key`はないが、腕力で何番目のデータが何を意味するのかは解析済…）

Goは構造体が`Unmarshaler`インターフェースを実装していればそれを使ってUnmarshalを行なう。

https://golang.org/pkg/encoding/json/#Unmarshaler
```go
type Unmarshaler interface {
        UnmarshalJSON([]byte) error
}
```

なので、`TotalsArray`に`UnmarshalJSON([]byte) error`を実装する。

# `[]interface{}`で受けて強引にパースしていく
`UnmarshalJSON([]byte) error`内で配列をパースしていく。
`totals`は複数型のデータを持っているため、単純な`[]int`のような配列では受けることができない。
そのため、一度`[]interface{}`で受けて、中身を型キャストして構造体に当てはめていく。`interface{}`オブジェクトとして`Unmarshal`すると、JSONデータは以下のように解釈される。

- encoding/json/#Unmarshal
  - https://golang.org/pkg/encoding/json/#Unmarshal

```
bool, for JSON booleans
float64, for JSON numbers
string, for JSON strings
[]interface{}, for JSON arrays
map[string]interface{}, for JSON objects
nil for JSON null
```

今回は`[]interface{}`の何番目がどんなデータかわかっているので、以下のようにタイプキャストして構造体のフィールドに取得した情報を代入していく。

```go
// UnmarshalJSON はエラーハンドリングを省略した簡易コード
func (ta *TotalsArray) UnmarshalJSON(data []byte) error {
	var arr interface{}
	// 引数として"totals"のデータが渡ってくる
	json.Unmarshal(data, &arr)

	// 0番目は数値(float64)であると決め打ち
	f, _ := arr[0].(float64)
	ta.Files = int(f)

  // 3番目は文字列(string)であると決め打ち
	c, _ := arr[3].(string)
	ta.CoverageRatio = c

	return nil
}
```

あとはこの`TotalsArray`を使ってJSONをUnmarshalすればよい。

```go
var got struct {
				Totals TotalsArray `json:"totals"`
			}

json.Unmarshal(data, &got)
```


# 終わりに
自分でAPIを設計するときはこんなJSONを出力するのは辞めよう。利用者がその値の意味を理解できない可能性があるし、配列内の順序が変わっていたとしても利用者は気づくことができないのでAPI間でバグが混入する可能性が高い。

# 参考
- JSON and Go
  - https://blog.golang.org/json-and-go
- encoding/json/#Unmarshal
  - https://golang.org/pkg/encoding/json/#Unmarshal

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B00LGJTXT8&linkId=8a84541627790149dbb0561368fbfafc"></iframe>
