+++
title = "Goのtestingを理解する in 2018 - iotestサブパッケージ編 #go"
date = 2018-09-09T21:51:11+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "test"]
+++
この記事は以下の記事で触れなかった`testing/iotest`について触れる。

- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018)

<!--more-->

# TL;DR
- testing/iotestパッケージ
  - https://golang.org/pkg/testing/iotest/
- `testing/iotest`パッケージは`io.Reader`/`io.Writer`のテスト用のラッパーを提供する
- エッジケースな挙動あるいはエラーを戻すラッパーと入出力をフックするラッパーが定義されている
- 入出力周りのテストヘルパーを書くときの参考にもなる

# func DataErrReader(r io.Reader) io.Reader
- https://golang.org/pkg/testing/iotest/#DataErrReader

`DataErrReader`は`Read(p []byte) (n int, err error)`メソッドを呼んだとき最後に`n != 0, err = io.EOF`を返す`io.Reader`オブジェクトを戻す。

- https://play.golang.org/p/e3eSeyirp_i

```go
	orign := []byte("Hello\nbyte.Reader\n")
	type want struct {
		n         int
		buf       string
		wantError error
	}
	tests := []struct {
		subject string
		r       io.Reader
		size    int
		wants   []want
	}{
		{
			"DataErrReader",
			iotest.DataErrReader(bytes.NewReader(orign)),
			5,
			[]want{
				{5, "Hello", nil},
				{5, "\nbyte", nil},
				{5, ".Read", nil},
				{3, "er\n\x00\x00", io.EOF}, // return with io.EOF
			},
		},
	}

	for _, tt := range tests {
		t.Run(tt.subject, func(t *testing.T) {
			for _, want := range tt.wants {
				buf := make([]byte, tt.size)
				size, err := tt.r.Read(buf)
				if size != want.n {
					t.Fatalf("want %d, but got = %d\n", want.n, size)
				}
				if string(buf) != want.buf {
					t.Fatalf("want %#v, but got = %#v\n", want.buf, string(buf))
				}
				if err != want.wantError {
					t.Fatalf("want io.EOF, but got = %#v\n", err)
				}
			}
		})
	}
```

通常、`io.Reader`インターフェースの実装には、`err == io.EOF`のとき`n = 0`かつその前の読み込みで全てのデータの読み込みが終了していることを想定するだろう。  
`DataErrReader`でラップした`io.Reader`のオブジェクトは読み込みが終わった時、`io.EOF`と読み込んだデータを同時に返す。  
「そのような実装を想定しないといけないことがあるのか？」というと、`http.Get`がそのような動きをするらしい。

- Go言語のHTTPリクエストのレスポンスボディーとEOF
  - https://itchyny.hatenablog.com/entry/2017/12/12/170000

# func HalfReader(r io.Reader) io.Reader
https://golang.org/pkg/testing/iotest/#HalfReader

- https://play.golang.org/p/7OBG8uUJu1B

```go
	orign := []byte("Hello\nbyte.Reader\n")
	type want struct {
		// DataErrReaderのサンプルコードと同じなので省略
	}
	tests := []struct {
		// DataErrReaderのサンプルコードと同じなので省略
	}{
		{
			"HalfReader",
			iotest.HalfReader(bytes.NewReader(orign)),
			5,
			[]want{
			  // len(5)のbufferでReadしても、半分の3バイトしか読み込んでくれない
				{3, "Hel\x00\x00", nil},
				{3, "lo\n\x00\x00", nil},
				{3, "byt\x00\x00", nil},
				{3, "e.R\x00\x00", nil},
				{3, "ead\x00\x00", nil},
				{3, "er\n\x00\x00", nil},
				{0, "\x00\x00\x00\x00\x00", io.EOF},
			},
		},
	}

	for _, tt := range tests {
		t.Run(tt.subject, func(t *testing.T) {
			// DataErrReaderのサンプルコードと同じなので省略
		})
	}
}

```

# func OneByteReader(r io.Reader) io.Reader
- https://golang.org/pkg/testing/iotest/#OneByteReader

`OneByteReader`はその名の通り、常に1バイトしか読みこまない`io.Reader`を返す。

- https://play.golang.org/p/b-GVUag2eSN
```go
	orign := []byte("Hello\nbyte.Reader\n")
	type want struct {
		// DataErrReaderのサンプルコードと同じなので省略
	}
	tests := []struct {
		// DataErrReaderのサンプルコードと同じなので省略
	}{
		{
			"OneByteReader",
			iotest.OneByteReader(bytes.NewReader(orign)),
			5,
			[]want{
				// 1バイトしか読み込まない
				{1, "H\x00\x00\x00\x00", nil},
				{1, "e\x00\x00\x00\x00", nil},
				{1, "l\x00\x00\x00\x00", nil},
				{1, "l\x00\x00\x00\x00", nil},
				{1, "o\x00\x00\x00\x00", nil},
				{1, "\n\x00\x00\x00\x00", nil},
				{1, "b\x00\x00\x00\x00", nil},
				{1, "y\x00\x00\x00\x00", nil},
				{1, "t\x00\x00\x00\x00", nil},
				{1, "e\x00\x00\x00\x00", nil},
				{1, ".\x00\x00\x00\x00", nil},
				{1, "R\x00\x00\x00\x00", nil},
				{1, "e\x00\x00\x00\x00", nil},
				{1, "a\x00\x00\x00\x00", nil},
				{1, "d\x00\x00\x00\x00", nil},
				{1, "e\x00\x00\x00\x00", nil},
				{1, "r\x00\x00\x00\x00", nil},
				{1, "\n\x00\x00\x00\x00", nil},
				{0, "\x00\x00\x00\x00\x00", io.EOF},
			},
		},
	}

	for _, tt := range tests {
		t.Run(tt.subject, func(t *testing.T) {
			// DataErrReaderのサンプルコードと同じなので省略
		})
	}
```

# func TimeoutReader(r io.Reader) io.Reader
- https://golang.org/pkg/testing/iotest/#TimeoutReader

`TimeoutReader`は二回目の`Read`呼び出し時にエラーが発生する。返されるエラーは`iotest`パッケージ内に定義されている。


- https://play.golang.org/p/dnwZwG5NyzG
```go
	orign := []byte("Hello\nbyte.Reader\n")
	type want struct {
		// DataErrReaderのサンプルコードと同じなので省略
	}
	tests := []struct {
		// DataErrReaderのサンプルコードと同じなので省略
	}{
		{
			"TimeoutReader",
			iotest.TimeoutReader(bytes.NewReader(orign)),
			5,
			[]want{
				{5, "Hello", nil},
				{0, "\x00\x00\x00\x00\x00", iotest.ErrTimeout}, // ErrTimeout on the second read with no data.
			},
		},
	}

	for _, tt := range tests {
		t.Run(tt.subject, func(t *testing.T) {
			// DataErrReaderのサンプルコードと同じなので省略
		})
	}
```

# func TruncateWriter(w io.Writer, n int64) io.Writer
- https://golang.org/pkg/testing/iotest/#TruncateWriter


`TruncateWriter`は第二引数で指定されたバイト数だけしか`w`に書き込こまない`io.Writer`オブジェクトを返す。
一定以上書き込めない出力先を生成するのに利用できる。

- https://github.com/golang/go/blob/50bd1c4d4eb4fac8ddeb5f063c099daccfb71b26/src/archive/tar/writer_test.go#L487

- https://play.golang.org/p/GvFtBx2rPcu
```go
	func TestTruncateWriter(t *testing.T) {
	orign := []byte("Hello\nbyte.Reader\n")
	wants := []struct {
		buf string
		n   int
	}{
		{"Hel", 3},
		{"Hello\nby", 8},
	}

	for _, want := range wants {
		b := &bytes.Buffer{}
		w := iotest.TruncateWriter(b, int64(want.n))
		w.Write(orign)

		if got := b.Bytes(); string(got) != want.buf {
			t.Fatalf("want %#v, but got = %#v\n", want.buf, string(got))
		}
	}
}
```

# func NewWriteLogger(prefix string, w io.Writer) io.Writer
- https://golang.org/pkg/testing/iotest/#NewWriteLogger
- https://golang.org/pkg/testing/iotest/#NewReadLogger

`NewWriteLogger`/`NewReadLogger`は書き込んだ(読み込んだ)結果をログ出力にフックする`io.Writer`(`io.Reader`)を返す。

たとえば、先ほどの`TestTruncateWriter`テストに`NewWriteLogger`を挟むと、

```go
		w = iotest.NewWriteLogger("Hook write", w)
```

`io.Writer`に書き込んだタイミングでその内容を16進数でログに出力する。

```bash
go test ./iotest -v -run TestTruncateWriter
=== RUN   TestTruncateWriter
2018/09/09 21:48:13 Hook write 48656c6c6f0a627974652e5265616465720a
2018/09/09 21:48:13 Hook write 48656c6c6f0a627974652e5265616465720a
--- PASS: TestTruncateWriter (0.00s)
PASS
ok  	github.com/budougumi0617/go-testing/iotest	0.009s
```


# 参考
- Go言語のHTTPリクエストのレスポンスボディーとEOF
  - https://itchyny.hatenablog.com/entry/2017/12/12/170000

# 関連
- [Goのtestを理解する in 2018 #go](/2018/08/19/go-testing2018)
- [Goのtestingを理解する in 2018 - Examples編 #go](/2018/08/30/go-testing2018-examples/)
- [Goのtestingを理解する in 2018 - quickサブパッケージ編 #go](/2018/09/05/go-testing2018-quick)

