+++
title= "[Go] 構造体オブジェクト初期化時にフィールド名を指定することを強制する #golangjp"
date= 2019-08-17T08:53:45+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["golang"]
keywords = ["Go", "golang"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++


静的解析に頼らず、コンパイル時に構造体オブジェクトの初期化でフィールド名の指定を強制するためのTips。

<!--more-->

# TL;DR
- 構造体初期化時、フィールド名を省略することができる
    - `p := &mypkg.Person{"alice", 12}`
- GoのComposite literals
    - https://golang.org/ref/spec#Composite_literals
- 構造体に非公開フィールドを含めることでフィールド名を明示的に指定しないとComposite literalsが使えなくなる
- 非公開フィールドは`struct{}`で宣言すればメモリサイズも増えない
    - `reflect.TypeOf(struct{}).Size() // 0`

# Goにおける構造体の初期化方法
Goでは構造体を初期化するときに`new`(`map`などの場合は`make`)を使う他にComposite literalsを使うことができる。  
このとき、フィールド名を指定せずに値を指定していた場合は、構造体のフィールド定義順に初期化されていく。

```go
type Person struct {
    Name string
    Age  int
}

func foo(){
    p1 := Person{"alice", 10}
    p2 := Person{Name:"bob", Age:20}
}
```


# 構造体に非公開フィールドを含めることでフィールド名を明示的に指定しないとComposite literalsが使えなくなる
[Struct literalsに関する仕様][spec]は2019/08/17時点では以下のように書かれている。

[spec]:https://golang.org/ref/spec#Composite_literals

- A key must be a field name declared in the struct type.
- An element list that does not contain any keys must list an element for each struct field in the order in which the fields are declared.
- If any element has a key, every element must have a key.
- An element list that contains keys does not need to have an element for each struct field. Omitted fields get the zero value for that field.
- A literal may omit the element list; such a literal evaluates to the zero value for its type.
- It is an error to specify an element for a non-exported field of a struct belonging to a different

ここで、Composite literalsを使うときに以下の仕様を利用することで、Composite literalsによる初期化を行なうとき、フィールド名の指定を強制することができる。

- フィールド名を指定しない場合、すべてのフィールドの初期値を指定しないといけない
- パッケージ外で非公開フィールドの値は指定できない

https://play.golang.org/p/fql-HWgBMfC

```go
-- go.mod --
module sample

go 1.12
-- sub/sub.go --
package sub

type MustKey struct {
	Name   string
	Second string
	_hoge  struct{}
}
-- main.go --
package main

import (
	"fmt"
	"sample/sub"
)

func main() {
	// mk := sub.MustKey{"hoge", "bar"} // ./main.go:9:28: too few values in sub.MustKey literal
	mk := sub.MustKey{}
	fmt.Printf("%v\n", mk)
}

```

# 非公開フィールドは`sturct{}`を使えばメモリサイズも増えない
このような実装は公式パッケージ内でも行われている。

- [database/sql/sql.go#L81-L96][sqlgo81]

[sqlgo81]:https://github.com/golang/go/blob/0212f0410f845815f5327a7f2e705891a9598f3d/src/database/sql/sql.go#L81-L96

```go
type NamedArg struct {
	_Named_Fields_Required struct{}

	// Name is the name of the parameter placeholder.
	//
	// If empty, the ordinal position in the argument list will be
	// used.
	//
	// Name must omit any symbol prefix.
	Name string

	// Value is the value of the parameter.
	// It may be assigned the same value types as the query
	// arguments.
	Value interface{}
}
```

チャネル利用時でも使われる`struct{}`型（空構造体）はサイズが`0`なので、パフォーマンス的にも悪影響はない。
同パッケージ内では、次のサンプルコードのようにフィールド名を指定せずに初期化できるが、さすがに`struct{}{}`とまで書いて初期化するならば気づくだろう…

- https://play.golang.org/p/DriUkPxU_EQ

```go
package main

import (
	"fmt"
	"reflect"
)

type MustKey struct {
	Name   string
	Second string
	_hoge  struct{}
}

func main() {
	mk := MustKey{"hoge", "bar", struct{}{}}
	fmt.Printf("%d\n", reflect.TypeOf(mk).Size()) // 20
	fmt.Printf("%d\n", reflect.TypeOf(mk._hoge).Size())  // 0
}
```

# 終わりに
可読性向上・実装ミス軽減につながるので構造体初期化時は必ずフィールド名を指定するようにしている。
が、永続化データと対応する構造体に非公開フィールドを用意することはないので、コンパイルエラーが出るからしていたわけではなかった。  

（`http.Request`など、）公式パッケージの構造体を使うときに無意識に使っていたのだろうが、改めて勉強になった。
今回は[@podhmo](https://twitter.com/podhmo)さんのtweetで見かけたのでちゃんと仕様まで読んでみた。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">そういえば、goのdatabase/sqlでNamedArgのフィールドに謎の値が含まれているのなるほどなーと思ったりした。<br>（たぶんフィールド名指定しない表現でのリテラルを禁止している）<a href="https://t.co/3gBixNUUmi">https://t.co/3gBixNUUmi</a></p>&mdash; po (@podhmo) <a href="https://twitter.com/podhmo/status/1160550001791000576?ref_src=twsrc%5Etfw">August 11, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

