+++
title= "[Go] context.TODO()を使って漸進的にcontext対応を始める"
date= 2020-02-21T09:55:37+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["golang", "context"]
keywords = ["Go", "Go言語", "context", "context.Context"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goではメソッドや関数の引数に`context.Context`が含められていると何かと便利だ。  
とはいえ、最初からアプリケーションが`context.Context`を考慮していない場合もある。  
アプリケーションを漸進的に`context.Context`に対応させる方法を書いておく。

<!--more-->

# TL;DR
- キャンセル通知や透過的な情報をやりとりするための仕組みが`context.Context`
    - ある操作のキャンセルを親gorotuineから伝える
    - リクエストIDなどをネストしたメソッドに伝える（透過的な情報しか含めてはいけない！）
- 運用状態のアプリケーションの各メソッドを全て`context.Context`に対応させるのは大変
- `context.TODO()`を使って少しずつ始めよう
    - https://golang.org/pkg/context/#TODO
- `context`パッケージの思想を知るにはThe Go Blogか、「Go言語による並行処理」を読むといいだろう。
    - https://blog.golang.org/context
    - https://www.amazon.co.jp/dp/4873118468

# contextパッケージとは
Goは`context`パッケージというキャンセル意思や特定のデータを透過的に呼び出し先の関数に伝えるための仕組みがある。  
具体的にいうと、`goroutine`による並行処理中の実行停止に利用することができる。標準パッケージやメジャーな3rdパッケージはほぼ対応している。  
また、APM（Application Perfomance Monitoring）やエラー通知サービスにデータを送信するさい、リクエストIDを含めたりしたいときがあるだろう。  
`context.Context`は、`WithValue`メソッドを使ってそのような本来そのメソッドのロジックに無関係の（トレースなどに利用した）透過的なデータを内包させることができる。  
詳しい説明はパッケージドキュメントやThe Go Blogを読めばよいだろう。

- `context`パッケージ
    - https://golang.org/pkg/context/
- Go Concurrency Patterns: Context | The Go Blog
    - https://blog.golang.org/context

透過的なデータの定義（決定指針）は「4.12 contextパッケージ」の章がわかりやすい。  
（Goの辞書であるプログラミング言語GoはGo1.6時代の本なので、残念ながらGo1.7で追加された`context`に関する解説はない）

- Go言語による並行処理
    - https://www.amazon.co.jp/dp/4873118468

# context.Contextに対応させる
では、実際に`context.Context`を利用するメソッドにリファクタリングしていくにはどうしたらよいだろうか。
最初は引数に`context.Context`を含めるだけから始めればよいだろう。慣例的に、`context.Context`は第一引数にする。
（`context.Contextが第一引数になっていない`と警告するlinterも存在する）

ここは単純にメソッドの引数を変更していくだけだ。

```diff
 // CompanyRepository is the repository to get company resource.
 type CompanyRepository interface {
-       Get(CompanyID) (domain.Company, error)
+       Get(context.Context, CompanyID) (domain.Company, error)
 }
```

API呼び出し(`http.Request`構造体)や`database/sql`パッケージを使ったDB操作を行なっている場合は、その生成、呼び出しも`context.Context`に対応させると良いだろう。  
`context.Context`を新しく引数に加えても、（呼び出し時に与えた`context.Context`のキャンセル処理などを実施しなければ、）何も挙動に影響を与えない。  
一通り修正を終えた後に実際に`context.Context`を使ったログ生成や、キャンセル処理の実装を加えていけばよいだろう。

```diff
-func (c *CompanyServiceClient) Get(cid CompanyID) (domain.Company, error) {
+func (c *CompanyServiceClient) Get(ctx context.Context, cid CompanyID) (domain.Company, error) {
        url := c.URL + "/companies"

-       req, err := http.NewRequest("GET", url, nil)
+       req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
        if err != nil {
                // ...
```

Go1.13からは`http.Request`を生成する際に、`context.Context`を含んだ`NewRequestWithContext`関数を利用できるようになっている。  
`database/sql`パッケージもGo1.8から`Query`メソッドの引数に`context.Context`を含んだ`QueryConntext`メソッドなどが用意されている。

- `http.NewRequestWithContext`
    - https://golang.org/pkg/net/http/#NewRequestWithContext
- `database/sql`パッケージ
    - https://golang.org/pkg/database/sql/

あとはこのリファクタリングを他のメソッドにも適用していくのだが、業務で行なっているアプリケーションのリファクタリングを一気に行うのは無理だろう。  
では、途中まで対応したとき、`context.Context`を引数にしためメソッドを呼び出すための`context.Context`はどこから用意すればよいのだろうか。

# 暫定的にcontext.Contextを引数に与える場合は、context.TODO()を利用する。
`context.Context`を生成するとき、通常は上位から受け取った`context.Context`を利用するか、`context.Background()`関数で`context.Context`を生成する。

- `context.Background()`
    - https://golang.org/pkg/context/#Background

しかし、`context`パッケージにはこのようなときのために利用する、`context.TODO()`関数が用意されている。

```diff
 // ListEmployees gets employees in comapany.
 func (e *EmployeesService) ListEmployees(cid CompanyID) (domain.Company, *ErrorResult) {
-       company, err := s.CompanyRepository.Get(cid CompanyID)
+       company, err := s.CompanyRepository.Get(context.TODO(), cid CompanyID)
        if err != nil {

```

`context.TODO()`で生成される`context.Context`は関数名のとおり、一時的な`context.Context`である。
GoDocにも「どの`context.Context`を使うかわからないとき、他の関数が対応していなくて`context.Context`が用意できないときに使ってね。」のように記載されている。

- `context.TODO()`
    - https://golang.org/pkg/context/#TODO

> TODO returns a non-nil, empty Context. Code should use context.TODO when it's unclear which Context to use or it is not yet available (because the surrounding function has not yet been extended to accept a Context parameter).

あとは少しずつ`context.Cntext`を引数にとるメソッドを増やしていけば良い。


# 終わりに
`context.Context`の使い方ではなく、`context.Context`を使うための準備方法をまとめた。  
`context.Context`が用意されていると、並行処理を挟むようになったとき、リクエストIDやトレーシングデータをSentryなどのエラー通知サービスやAPMへのデータを送信することが非常に簡単になる。  
新しく作るアプリケーションの場合は、ひとまず対応しておいたほうが良いだろう（必ず必要になるときがくる）。  
`context.Context`対応を終えたアプリケーションでどのように`context.Context`を利用していくのかもいずれまとめたい。

# 参考文献
- [Go Concurrency Patterns: Context | The Go Blog](https://blog.golang.org/context)
- [Go言語による並行処理](https://www.amazon.co.jp/dp/4873118468)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873118468&linkId=727ff07e1319c560674f5751393013cb"></iframe>
