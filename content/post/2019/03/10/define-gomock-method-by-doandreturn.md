+++
title= "[Go][gomock]引数によって挙動を変えるモックメソッドを定義する #golangjp"
date= 2019-03-10T12:53:08+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "gomock", "test"]
keywords = ["Go", "test", "gomock", "mock", "モック"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

Goのテストでモックを使う場合、大抵の場合はgomockを使うと思う。  
gomockでモックメソッドの挙動を宣言するときのTipsをメモしておく。

- https://github.com/golang/mock/
- https://godoc.org/github.com/golang/mock/gomock

<!--more-->

# TL;DR
- `gomock`で作ったモックのメソッドの戻り値を定義するときは`Return`を使う
  - https://godoc.org/github.com/golang/mock/gomock#Call.Return
- メソッドの引数によって戻り値を変えたいときは`DoAndReturn`を使う
  - https://godoc.org/github.com/golang/mock/gomock#Call.DoAndReturn
- テストケースごとに柔軟にモックの挙動を変えたいときはテストケースにセットアップ関数を仕込む

なお、文中のサンプルコードのベースは以下にある。

- [https://github.com/budougumi0617/til/go/gomock](https://github.com/budougumi0617/til/tree/8dc39e8a8100d67b9f6e6e1ae845b165d7a494ed/go/gomock)

## 文中のサンプルコードについて
文中のサンプルコードで利用しているモックは以下のインターフェースのモックである。

```go
type Client interface {
	Method(in string) (string, error)
}
```

モックは以下のコマンドにより自動生成している。
```bash
$ mockgen -destination mock/client.go \
  -package mock github.com/budougumi0617/til/go/gomock Client
```

実際に生成されたモックコードの宣言の抜粋が以下のコードになる。

```go
package mock

// ...

// MockClient is a mock of Client interface
type MockClient struct {
	// ...
}

// NewMockClient creates a new mock instance
func NewMockClient(ctrl *gomock.Controller) *MockClient {
	// ...
}
```

# モックを使ってメソッドの戻り値を返す

`gomock`戻り値を返すメソッドのモックは次のように書く。

```go
// モックオブジェクトを生成
m := mock.NewMockClient(gomock.NewController(t)) // t *testing.T)

// Client.Methodメソッドの戻り値を定義する。mock.Any()を使って、引数の値は問わない。
m.EXPECT().Method(mock.Any()).Return("hoge", nil)
```

# 引数によってメソッドの戻り値が変わるようにする
テストを書くときは大抵の場合、あるテスト対象の処理に対して複数のテストケースを書くことになる。そのとき、テストケース一つ一つでモックを再定義するのは面倒だ。ひとつのモック定義に対して挙動が異なるモックのロジックを仕込みたくなる。  
ここで、渡ってくる引数によって条件を変えたい場合は`DoAndReturn`メソッドを使って定義する。  
`DoAndReturn`メソッドには、モックするメソッドと同じシグネチャの無名関数を渡す。  
無名関数内は自由な処理を書くことができるので、これでモックメソッドに任意の挙動を実装しておくことができる。


```go
func TestDoAndReturn2(t *testing.T) {

	in := "any value"

	ctrl := gomock.NewController(t)
	defer ctrl.Finish()
	mc := mock.NewMockClient(ctrl)

	mc.EXPECT().Method(in).DoAndReturn(
	  // エラーを返すモックメソッドの実装
		func(in string) (string, error) {
			return "", fmt.Errorf("%s", in)
		})

	if _, err := mc.Method(in); err == nil {
		t.Error("cannot get error")
	}

}
```

例えば`switch`条件で引数によって挙動を変える例は以下になる。

```go
func TestDoAndReturn3(t *testing.T) {

	errin := "raise error"
	specify := "specify"

	ctrl := gomock.NewController(t)
	defer ctrl.Finish()
	mc := mock.NewMockClient(ctrl)

	mc.EXPECT().Method(gomock.Any()).DoAndReturn(
		func(in string) (string, error) {
			switch in {
			case errin:
				return "", fmt.Errorf("%s", in)
			case specify:
				return "!!!!!", nil
			}
			return in, nil
		}).AnyTimes() // By default, method is called only once.

	if _, err := mc.Method(errin); err == nil {
		t.Error("cannot get error")
	}

	in := "hogehoge"
	if got, _ := mc.Method(in); got != in {
		t.Errorf("want %s, but got %s\n", in, got)
	}

	if got, _ := mc.Method(specify); got != "!!!!!" {
		t.Errorf("want %s, but got %s\n", "!!!!!", got)
	}
}
```

# テストごとにモックのメソッドの宣言を変える
そもそもテストケースごとに設定する条件が全く異なるときもある。

- テストケースごとにモックメソッドの飛び出し回数が異なる(`Times`で呼び出し回数を指定したい)
- あるテストケースのときだけメソッドAにモックの挙動を仕込みたい

このようなとき、私はサブテストの条件の中にモックの挙動をセットアップする関数を定義している（テストの条件に関数を含めるのには是非があるのは承知の上で）。  
テストケースの冒頭でセットアップ関数を呼ぶようにしておけばよい。以下の例ではテスト条件のテーブルに`setClient func(*mock.MockClient, string)`というモックのセットアップ関数を定義した例だ。

```go
func TestBySubTest(t *testing.T) {
	tests := []struct {
		name string
		// Inject rule to mock.
		setClient func(*mock.MockClient, string)
		in        string
		want      string
		wantErr   bool
	}{
	  // モックのセットアップ関数以外の初期化宣言は省略している
		{
			setClient: func(mc *mock.MockClient, in string) {
				mc.EXPECT().Method(in).Return("hoge", nil)
				mc.EXPECT().Method(in).Return("bar", nil)
			},
		},
		{
			setClient: func(mc *mock.MockClient, in string) {
				mc.EXPECT().Method(in).DoAndReturn(
					func(in string) (string, error) {
						return in, nil
					})
			},
		},
		{
			setClient: func(mc *mock.MockClient, in string) {
				mc.EXPECT().Method(in).DoAndReturn(
					func(in string) (string, error) {
						return "", fmt.Errorf("%s", in)
					})
			},
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			ctrl := gomock.NewController(t)
			defer ctrl.Finish()

			mc := mock.NewMockClient(ctrl)
			tt.setClient(mc, tt.in)

			// テスト内容を書く。
	}

}
```

# 終わりに
`DoAndReturn`メソッドを使ったモックの書き方の記事がなかったので、いくつかのパターンも含めて書いてみた。  
ただ、モックを使っているとテストフェイルしたときのエラー出力なども複雑になる。そもそもあまりモックに頼らないような仕組みの提供や実装にしておくことも大事だと思う。


