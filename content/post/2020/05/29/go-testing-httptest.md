+++
title= "Goのtestを理解する - httptestサブパッケージ編"
date= 2020-05-29T12:19:49+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","test"]
keywords = ["Go","testing","http","httptest"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goのテストについていくつかまとめを書いていた。

- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018)
- [Goのtestを理解する in 2019](/2019/10/30/go-testing2019/)

触れるのを忘れていたhttptestパッケージについてまとめる。

- https://golang.org/pkg/net/http/httptest/

<!--more-->

# TL;DR
- net/http/httptestパッケージ
  - https://golang.org/pkg/net/http/httptest/
- `net/http/httptest`パッケージはサーバ/クライアント両方のHTTP周りのテストで使う
- サーバのHTTPハンドラーのテストを書くとき
    - https://golang.org/pkg/net/http/httptest/#NewRequest
    - https://golang.org/pkg/net/http/httptest/#NewRecorder
- ダミーサーバを立ててクライアントのテストを書くとき
    - https://golang.org/pkg/net/http/httptest/#NewServer
    - https://golang.org/pkg/net/http/httptest/#NewTLSServer

# net/http/httptestパッケージ
`net/http/httptest`パッケージはHTTPのテストを書くのに便利なパッケージだ。
サーバのHTTPハンドラーを書くときは次の2つを利用してテストを書く。
- https://golang.org/pkg/net/http/httptest/#NewRequest
- https://golang.org/pkg/net/http/httptest/#NewRecorder

クライアント側の動作確認をしたくて、テストコードでダミーのサーバを立ち上げたいときは次のいずれかを使う。
- https://golang.org/pkg/net/http/httptest/#NewServer
- https://golang.org/pkg/net/http/httptest/#NewTLSServer

# サーバのHTTPハンドラーのテストコードを書く
Goでwebアプリケーションを書くとき、WebフレームワークやgRPCなどを利用していない場合は、`http.HandleFunc`型のシグネチャでハンドラー関数を書くだろう。

- https://golang.org/pkg/net/http/#HandleFunc

```go
h := func(w http.ResponseWriter, r *http.Request) {
  io.WriteString(w, "Hello from handler!\n")
}

http.HandleFunc("/", h)
log.Fatal(http.ListenAndServe(":8080", nil))
```

このようなハンドラーのテストを書くときにわざわざHTTPサーバを立てる、というのは少々めんどくさい。
こんなときに利用できるのが`httptest.NewRequest`と`httptest.ResponseRecorder`だ。

- https://golang.org/pkg/net/http/httptest/#NewRequest
- https://golang.org/pkg/net/http/httptest/#NewRecorder

`httptest.NewRequest`は擬似的なHTTPのリクエストを作成できる。生成関数以外は通常の`http.Request`と同じ方法でHTTPヘッダーなどを組み立てられる。  
`httptest.NewRecorder`は`http.ResponseWriter`を満たす`*httptest.ResponseRecorder`オブジェクトを取得できる。
このオブジェクトを利用してHTTPハンドラーの戻り値を検証するテストコードを書ける。

具体的には利用するときは次のようなテストコードになるだろう。

```go
// テスト対象のHTTPハンドラー
func myHandler(w http.ResponseWriter, r *http.Request) {
  io.WriteString(w, "Hello from handler!\n")
}

func TestMyHandler(t *testing.T) {
  // Arrange
  reqBody := bytes.NewBufferString("request body")
  req := httptest.NewRequest(http.MethodGet, "http://dummy.url.com/user", reqBody)

  // 生成後は*http.Requestオブジェクトと同じように扱える
  q := req.URL.Query()
  q.Add("", tt.args.code)
  req.URL.RawQuery = q.Encode()

  // レスポンスを受け止める*httptest.ResponseRecorder
  got := httptest.NewRecorder()

  // Act
  myHandler(got, req)
  
  // Assertion
  // http.Clientなどで受け取ったhttp.Responseを検証するときとほぼ変わらない
  if got.Code != http.StatusOk {
      t.Errorf("want %d, but %d", tt.wantStatus, got.Code)
  }
  // Bodyは*bytes.Buffer型なので文字列の比較は少しラク
  if got := got.Body.String(); got != tt.wantBody {
    t.Errorf("want %s, but %s", tt.wantBody, got)
  }
  // http.Responseオブジェクトとしても比較できる。
  if resp := got.Result().Cookies(); resp.ContentLength == 0 {
    t.Errorf("resp.ContentLength was 0")
  }
}
```

# クライアントのテストコードを書くとき
HTTPクライアント側のテストを書くときに便利なのが、`*httptest.Server`オブジェクトだ。
実際のHTTP通信を使ったHTTPクライアント側のテストコードを書こうと思うと、次のようなテストコードを書く必要がある。

1. 通信先のサーバを模すダミーハンドラーを用意する
1. ダミハンドラーでリッスンする`*http.Server`をgoroutineで起動する
1. 別goroutine上でサーバがリッスンを開始するまで待機する
    - 起動前に通信をしてしまうとテストが失敗する
1. テスト対象のHTTPクライアントを初期化して、実行する
1. HTTPクライアントの実行結果を検証する
1. テスト終了時にダミハンドラーでリッスンするサーバを適切に週有量する

別goroutine上でダミーのサーバを動かして適切に待機、終了させるコードを毎回書くのは面倒だ。  
ここで、`*httptest.Server`オブジェクトを利用する。このオブジェクトが便利なのは内部でgoroutineを起動してリッスンを開始してくれる点だ。
また、`NewServer`関数から制御が戻ってきた時点でサーバがリッスンを開始しているため、クライアントが通信を開始するタイミングをあわせる必要もない。

`*httptest.Server`オブジェクトはHTTP通信なのか、HTTPS通信によって初期化関数を使い分ける。
- https://golang.org/pkg/net/http/httptest/#NewServer
- https://golang.org/pkg/net/http/httptest/#NewTLSServer

コレを使ってHTTPクライアントのテストを書くと、goroutineを意識せずにシンプルなテストコードを書くことができる。

```go
func TestClient(t *testing.T) {
  // Arrange
  // ServeMuxオブジェクトなどを用意してルーティングしてもよい
  h := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello, client")
  }
  // 別goroutine上でリッスンが開始される
  ts := httptest.NewServer(h)
  defer ts.Close()

  cli := &http.Client{}
  req, err := http.NewRequestWithContext(context.TODO(), "GET", ts.URL, strings.NewReader(""))
  if err != nil {
    t.Errorf("NewRequest failed: %v", err)
  }

  // Act
  resp, err := cli.Do(req)
  if err != nil {
    t.Fatal(err)
  }

  // Assertion
  got, err := ioutil.ReadAll(resp.Body)
  if err != nil {
    t.Fatal(err)
  }
  res.Body.Close()
  want := "Hello, client\n"
  if got != want {
      t.Errorf("want %q, but %q", want, got)
  }
}
```

待ち合わせのコードやgoroutineを起動するようなコードも書かず、シンプルなHTTPクライアントのテストを書くことができた。

# 終わりに
`httptest`パッケージを使うと、サーバのコードを書くときサーバ本体の実装を待たずともハンドラーごとのテストを書けて非常に便利だ。  
また、ダミーサーバはクライアントの実装だけでなく、サードパーティへの通信を含んだ処理のエンドポイントテストを書くときにも利用できる。  
テストの書きやすさはTDDやエンドポイントテストをやるためのモチベーションにもつながるので、このようなテストフレームワークが標準pkgとして公開されているのはとてもありがたい。

# 参考
- https://golang.org/pkg/net/http/httptest/

# その他のテスト関連の記事
- [Goのtestを理解する in 2019](/2019/10/30/go-testing2019/)
- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018)
- [Goのtestingを理解する in 2018 - Examples編 #go](/2018/08/30/go-testing2018-examples/)
- [Goのtestingを理解する in 2018 - quickサブパッケージ編 #go](/2018/09/05/go-testing2018-quick)
- [Goのtestingを理解する in 2018 - iotestサブパッケージ編 #go](/2018/09/09/go-testing2018-iotest/)
- [Goでサーバを立ち上げてE2Eテストを実施するCI用のテストコードを書く](/2020/03/27/http-test-in-go/)

