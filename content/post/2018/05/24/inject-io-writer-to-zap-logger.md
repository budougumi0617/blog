+++
title= "[Go]go.uber.org/zap.loggerの出力をbytes.Bufferに変更する #golang"
date= 2018-05-24T08:44:38+09:00
draft = false
slug = ""
categories = ["go"]
tags = ["golang","zap", "test"]

author = "budougumi0617"
+++

`go.uber.org/zap`の`zap.logger`は構造化されたログを高速に出力できるとしてGolangのLoggerの中で有名だ。

https://github.com/uber-go/zap

gRPCを用いたMicroservicesを構成する際にも利用されることが多い。

https://github.com/grpc-ecosystem/go-grpc-middleware/tree/master/logging/zap

`go.uber.org/zap`の`zap.logger`の利用方法としてよく見るのは
`zao.Config`構造体を生成し、`Build()`メソッドから`zap.Logger`インスタンスを生成するやり方だ。  
この生成方法だと、文字列で出力先を指定するため、一見標準(エラー)出力もしくはファイルにしか出力できないように読める。

https://godoc.org/go.uber.org/zap#Config

```go
    // OutputPaths is a list of paths to write logging output to. See Open for
    // details.
    OutputPaths []string `json:"outputPaths" yaml:"outputPaths"`
```

このログ出力先を`bytes.Buffer`(`io.Writer`)に変更する。

# TL;DR
- `zap.Config`の`Build()`メソッドで`zap.Logger`を生成するとログ出力先を標準(エラー)出力orファイルにしかできない
- `zap.New`関数で`zap.Logger`を生成すると任意の`io.Writer`にログを出力できる


```go
// 引数のbytes.Bufferにログを出力するzap.Loggerを生成する
func getDummyLogger(buf *bytes.Buffer) *zap.Logger {
        encoder := zapcore.NewConsoleEncoder(...)
        core := zapcore.NewCore(encoder, zapcore.AddSync(buf), zapcore.InfoLevel)
        return zap.New(core)
}
```

# zapとは

https://github.com/uber-go/zap

`go.uber.org/zap`は構造化したメッセージを高速にロギングできるライブラリ。
ただ、標準ライブラリの`log`パッケージのインターフェースと`zap.logger`構造体は大きく異なる。

# なぜやりたかったのか

一定の処理を加えたあとにログ出力をするlogging用のMiddlewareやInterceptorを作ろうと思ったときに、
`Logger`に渡すログ出力自体を検証したいと考えることがある。`Go`の場合、出力先が`io.Writer`インターフェースならば、テストをするときだけ`bytes.Buffer`などに置き換えて検証することが出来る。  
しかし、`zap.Config`にある`zap.Logger`を出力先を指定する設定方法は文字列指定であり、`stdout`(`stderr`)もしくは出力先のファイルパスしか設定出来ない。

https://godoc.org/go.uber.org/zap#Config

```go
    // OutputPaths is a list of paths to write logging output to. See Open for
    // details.
    OutputPaths []string `json:"outputPaths" yaml:"outputPaths"`
```

なので`zapcore`パッケージを使う方法で`io.Writer`へログ出力をする`zap.Logger`インスタンスを取得する。

# zap.New、zapcore.NewCoreを使ったzap.Loggerの生成
`zap.Logger`のインスタンスを生成する方法は`Config`インスタンスの`Build()`メソッドを利用する他に、`zap.New()`関数を利用する方法がある。

https://godoc.org/go.uber.org/zap#New

```go
func New(core zapcore.Core, options ...Option) *Logger
```

`New()`関数の`zapcore.Core`は`zapcore.NewCore`関数から生成できる。

https://godoc.org/go.uber.org/zap/zapcore#NewCore

```go
func NewCore(enc Encoder, ws WriteSyncer, enab LevelEnabler) Core
```

この第二引数の`zapcore.WriteSyncer`インターフェースが`zap.New()`関数から生成したときの`zap.Logger`の出力先になる。  
`zapcore.WriteSyncer`インターフェースは`io.Writer`と`Sync()`メソッドを満たしていればよい。

```go
type WriteSyncer interface {
    io.Writer
    Sync() error
}
```



そしてひとまずインターフェースを満たすだけで良いならば`io.Writer`を引数に`zapcore.WriteSyncer`を返す便利な関数が`zapcore`パッケージに存在する。

https://godoc.org/go.uber.org/zap/zapcore#AddSync

```go
func AddSync(w io.Writer) WriteSyncer
```

よって、以下のような手順を踏めば、`zapcore.NewCore`関数、`zap.New`関数を経由すれば`bytes.Buffer`(`io.Writer`)にログ出力をする`zap.Logger`を取得できる。

```go
func getDummyLogger(buf *bytes.Buffer) *zap.Logger {
        // ログ出力のフォーマットを指定できる
        encoder := zapcore.NewConsoleEncoder(zapcore.EncoderConfig{
                // TimeKey: "time", // 時刻情報は期待結果に含めにくいので省略
                NameKey:        "name",
                EncodeLevel:    zapcore.CapitalLevelEncoder,
                EncodeTime:     zapcore.ISO8601TimeEncoder,
                EncodeDuration: zapcore.StringDurationEncoder,
                EncodeCaller:   zapcore.ShortCallerEncoder,
        })
        core := zapcore.NewCore(encoder, zapcore.AddSync(buf), zapcore.InfoLevel)
        return zap.New(core)
}
```

# 終わりに
Goはシンプルな標準インターフェースとダックタイピングを提供している。  
そのため、疎な設計になっていれば標準使用のみで容易にモックを使うことが出来る。

