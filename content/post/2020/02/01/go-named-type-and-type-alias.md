+++
title= "[Go] Named typeとType aliasを使い分ける"
date= 2020-02-01T12:31:27+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["gotips","golang"]
keywords = ["Go", "Golang", "Go言語", "型エイリアス", "Named type"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


Goには既存の型に新しい名前をつける方法が2つある。

 - `type MyType int`と宣言するNamed type
 - `type MyType = int`と宣言するType alias

すでにいろいろ記事はあるものの、最近数回聞かれることがあったので改めてまとめておく。

<!--more-->

# Tl;DR
- Goには型に違う名前をつける方法がある。
    - Named typeとType alias
- Named typeを使うと完全に違う型として扱える
    - プリミティブな型に異なる型名をつけたり、メソッドを生やすこともできる
    - Value Object的な型を簡単に作ることができる
- Type aliasを使うと異なる名前だが同じ型として扱われる
    - リファクタリングをするときに使われる
    - 型付けを利用した使い分けをしたいならば向かない
- 基本的にNamed Typeを使えばよい。


なお、この記事はGo1.13を使って検証している（正確に言うと、2020/02/01時点のGo Playgroundで検証している）。


# Named type
Named typeはGo1.0からある機能だ。（正確な導入時期は知らないが、基本構文でできるので最初からありそうだ）。

ある型の定義をそのまま利用して、まったく違う型として認識される新しい型を定義できる。

- 別の型なので明示的にキャストしないと互換性がない
- 新しいメソッドを追加できる

以下のコードは`http.Request`型を使った新しい`MyRequest`型を宣言し、メソッドを追加している。

- https://play.golang.org/p/cG9o7W10IjF

```go
package main

import (
	"fmt"
	"net/http"
)

type MyRequest http.Request

func (mc MyRequest) MyFunc() {
	fmt.Println("in MyFunc", mc.Host)
}

func useMyRequest(mc MyRequest) {
	mc.MyFunc()
}

func main() {
	myReq := MyRequest{Host: "http://google.com"}
	useMyRequest(myReq)
}
```

Named typeはプリミティブな型に対しても利用することができる。  
Named typeで宣言した新しい型と元になった型には互換性がないため、Value Object的に利用することができる。

以下の例では　ユーザー名とメールアドレスの文字列の取り扱いを間違えている。

- https://play.golang.org/p/lX26gpsz21T

```go
package main

import "fmt"

type User struct {
	Name, Email string
}

func NewUser(name, email string) User { return User{Name: name, Email: email} }

func main() {
	name := "John Doe"
	mail := "example@foo.com"
	u := NewUser(mail, name) // 順番を間違えている。
	fmt.Println(u.Name) // example@foo.com
}
```

Named typeを使うことで、型チェックを利用して誤った使いかたを防ぐことができる。

- https://play.golang.org/p/1S7VT6lT69J

```go
package main

import "fmt"

type Name string
type MailAddress string

type User struct {
	Name  Name
	Email MailAddress
}

func NewUser(name Name, email MailAddress) User { return User{Name: name, Email: email} }

func main() {
	name := Name("John Doe")
	mail := MailAddress("example@foo.com")
	// cannot use mail (type MailAddress) as type Name in argument to NewUser
	// cannot use name (type Name) as type MailAddress in argument to NewUser
	u := NewUser(mail, name)
	fmt.Println(u.Name)
}

```

`type UserID int`、`type DocumentID int`のようにNamed typeを使っていけば、データベースまわりでIDの取り扱いをして間違えることもなくなる。  
また、メソッドを追加することもできるので、バリデーションなども追加しておくこともできる。

- https://play.golang.org/p/7e85wA1DPex

```go
package main

import (
	"fmt"
	"log"
)

type Password string

func (p Password) Validate() error {
	if len(p) < 16 {
		return fmt.Errorf("パスワードは16文字以上")
	}
	return nil
}

type User struct {
	Name     string
	Password Password
}

func NewUser(n string, pw Password) (*User, error) {
	if err := pw.Validate(); err != nil {
		return nil, err
	}
	return &User{n, pw}, nil
}

func main() {
	n := "John Doe"
	pw := Password("p@ssw0rd")
	u, err := NewUser(n, pw)
	if err != nil {
		log.Fatal(err) // パスワードは16文字以上
	}
	fmt.Println(u.Name)
}
```

# 型エイリアス（Type alias）
型エイリアスはGo1.9から追加された機能だ。

- [Changes to the language  | Go 1.9 Release Notes][release]

型エイリアスは主に特徴を持つ。

- キャストせずに同じ型として利用できる
- エイリアスに新しいメソッドは定義できない。

異なる型として利用できるようになる`Named type`と違い、型エイリアスを使った場合はまったく同じ型として利用できる。  
そのため、Named Typeのような型チェックを期待してもそれは行われないので注意する。

- https://play.golang.org/p/VgpS3x89-bO

```go
package main

import "fmt"

type Name = string // type alias
type MailAddress = string // type alias

type User struct {
	Name  Name
	Email MailAddress
}

func NewUser(name Name, email MailAddress) User { return User{Name: name, Email: email} }

func main() {
	name := Name("John Doe")
	mail := MailAddress("example@foo.com")
	u := NewUser(mail, name) // 型違いでコンパイルエラーにはならない！！！！！
	fmt.Println(u.Name) // example@foo.com
}
```

正直私は型エイリアスを使ったことがない。サードパーティ製のライブラリを使っていて、どうしても困るときがあったら出番なのかもしれない。

型エイリアスを使ったリファクタリングや型エイリアスが必要な背景は公式の記事や、 [@tenntenn](https://twitter.com/tenntenn) さんのQiitaの記事が詳しい。

- [Codebase Refactoring (with help from Go)](https://talks.golang.org/2016/refactor.article)
- [コンテキストの歴史にまつわるインタフェースの互換性と型エイリアス | Google App EngineでGoのバージョンアップを行う #golang #gae][qiita]

#  終わりに
Type aliasの説明は省略してしまったが、Named typeとType Aliasについてまとめた。  
Goは型をうまく使うことで安全なコーディングをすることができる。Named typeを使えばたった一行で新しい型が宣言でき、IDや文字列などの取り間違いを防ぐことができる。

# 参考
- [Changes to the language  | Go 1.9 Release Notes][release]
- [Codebase Refactoring (with help from Go)](https://talks.golang.org/2016/refactor.article)
- [Type declarations | The Go Programming Language Specification](https://golang.org/ref/spec#Type_declarations)
- [エキスパートGo by tenntenn| Slideshare](https://www.slideshare.net/takuyaueda967/go-77689475)
- [コンテキストの歴史にまつわるインタフェースの互換性と型エイリアス | Google App EngineでGoのバージョンアップを行う #golang #gae][qiita]

[release]: https://golang.org/doc/go1.9#language
[tenntenn]: https://twitter.com/tenntenn
[qiita]: https://qiita.com/tenntenn/items/757c249dad942b6326a9#%E3%82%B3%E3%83%B3%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88%E3%81%AE%E6%AD%B4%E5%8F%B2%E3%81%AB%E3%81%BE%E3%81%A4%E3%82%8F%E3%82%8B%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%95%E3%82%A7%E3%83%BC%E3%82%B9%E3%81%AE%E4%BA%92%E6%8F%9B%E6%80%A7%E3%81%A8%E5%9E%8B%E3%82%A8%E3%82%A4%E3%83%AA%E3%82%A2%E3%82%B9
