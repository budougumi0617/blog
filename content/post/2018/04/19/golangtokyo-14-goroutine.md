+++
title= "golang.tokyo #14 ゴールーチンとチャネルによる並行処理 参加メモ #golangtokyo"
date= 2018-04-19T09:41:43+09:00
draft = false
slug = ""
categories = ["go","report"]
tags = ["golang", "golangtokyo","goroutine"]
author = "budougumi0617"
+++

golang.tokyo #14はgroutineとchannelの話。  
@morkuniさんの発表は入門編、@knsh14さんの発表はdeep diveという感じで非常によかった。  
「golangは言語仕様が簡素だからすぐ覚えられる」とよく言われるけど、並行処理に関しては結構スキルがいるイメージ。  
なお、「並行処理」と「並列処理」の概念の違いはRob Pikeの以下の資料が分かりやすい。

**Concurrency is not Parallelism**  
https://talks.golang.org/2012/waza.slide#1

設計・実装するときにどんなことが起きるのか、どう動いているのか思い浮かぶのは重要なので、どちらの発表も非常にわかりやすくてよかった。  
LTではvgoの話を聞けたり今回も内容が盛りだくさんだった。  
以下、自分のメモ

|||
|---|---|
|URL|https://golangtokyo.connpass.com/event/82723/|
|会場|株式会社メルカリ|
|日時|2018/04/16(月) 19:10 〜 22:10|
|資料|[connpassイベント資料一覧](https://golangtokyo.connpass.com/event/82723/participation)|
|ハッシュタグ|[#golangtokyo](https://twitter.com/hashtag/golangtokyo)|
|Toggeter|https://togetter.com/li/1218725|

# ホリネズミでもわかるGoroutine入門 / golang.tokyo#14
株式会社メルカリ/ソウゾウ 森國 泰平 / [@morikuni](https://twitter.com/inukirom)  
https://speakerdeck.com/morikuni/golang-dot-tokyo-number-14

goroutine, chan, select, Wait系などのGoの平行処理に関する基本的な文法処理と注意点。

- goroutineの実行順は保証されない。
- `panic`したときは同じgoroutineの中にしか伝わらない。
- つまり、別のgoroutineで`panic`が起きたときは`defer`も動かないし、`recover`でcatch出来ない。
    - 例: https://play.golang.org/p/SwjvfKWQ8Aw

```go
package main

import (
	"fmt"
	"time"
)

func A() {
	B()
}

func B() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("Catch panic in B!")
		}
	}()
	go C()
	time.Sleep(3 * time.Second) // Heavy task...
}

func C() {
	defer func() {
		fmt.Println("defer in C!")
	}()
	D()
	time.Sleep(3 * time.Second) // Heavy task...
}

func D() {
	panic("dead")
}

func main() {
	A()
	fmt.Println("Hello, playground")
	time.Sleep(10 * time.Second) // Careless waiting...
}
```
- channelは読み込み、書き込み、読み書き可能なchannelを作れる
    - ただ単に合図を送りたい時は空`struct`で
    - 0バイトだから
- channelからのmessageは`select`で受け取る
    - ブロックしたくないときは`select`に`default`をつける
- goroutineあるある。goroutineが実行されない
    - Waitしていないとgoroutineが実行される前に`main`が終了している
    - `sync.WaitGroup`などを使ってWaitしよう
- groutineあるある。for rangeループでgoroutineを複数立ち上げて配列の処理をしたら何故か結果が全部一緒
    - 変数内容が毎回書き換えられているから。
    - (これはgoroutineあるあるというかfor rangeあるある？)
      - 「[プログラミング言語Go](https://amazon.jp/dp/4621300253) 5.6.1 警告：ループ変数の補足」を参照
    - `go func(e element){...}(e)`などとして関数の引数で受け取ろう
- goroutineあるある。goroutine中で呼び出し元から受け取ったスライスに`append`したら結果がぐちゃぐちゃ
    - 競合している。
    - そもそもgoroutine使う必要があるのか検討
    - 数値系は`sync/atomic`
    - `sync.Mutex`か`sync.RWMutex`使うのがベター
- goroutineあるある。全ての処理が未完了のまま終了してgoroutineリークする。
    - 複数goroutineの一部の結果だけ使って次に進むときは`context.Context`を使ってキャンセルしておく
    - （スライド中の`http.Get`を待機しているところは`case c <- result{res, err}:`って書かないとダメ(`:`が抜けてる)）
    - 例: https://play.golang.org/p/Ej1UGieAz77

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func task(ctx context.Context, s int, ch chan<- int) {
	time.Sleep(time.Duration(s) * time.Second) // Heavy task
	select {
	case ch <- s:
		fmt.Printf("Done task with %d\n", s)
	case <-ctx.Done():
		fmt.Printf("Cancel task with %d\n", s) // Write if cancel
	}
}

func main() {
	ch := make(chan int)
	ctx, cancel := context.WithCancel(context.Background())
	defer func() {
		cancel()
		time.Sleep(10 * time.Second) // Wait cancel result
	}()

	go task(ctx, 2, ch)
	go task(ctx, 10, ch)

	fmt.Printf("Return value %d\n", <-ch) // Wait result
}
```
- 便利ツール gotrace
    - https://github.com/divan/gotrace
    - traceファイルを作っておくと並行状態を可視化することができる
    - (チューニングしたgoの実行環境が必要なので）docker環境が必要
    - masterブランチは動かないっぽいので[go18](https://github.com/divan/gotrace/tree/go18)ブランチをチェックアウトする必要がある

スライドが発表を聞いていない人でもわかるようにまとまっているので、是非[スライドでみたほうが良い](https://speakerdeck.com/morikuni/golang-dot-tokyo-number-14)と思う。

自分はそこそこわかっているつもりだったが`context.Context`併用するサンプルコード書くらいでちょっとつまったので復習しないと…

# チャネルのしくみ
KLab 株式会社 鎌田健史 / [@knsh14](https://github.com/knsh14)  
https://speakerdeck.com/knsh14/tiyanerufalseshi-zu-mi

- ぜひやってほしいこと
    - GopherConのKavya-sanの発表を調べてみる
        - [[Youtube]GopherCon 2017: Kavya Joshi - Understanding Channels](https://www.youtube.com/watch?v=KBZlN0izeiY)
        - [github.com/gophercon/2017-talks/tree/master/KavyaJoshi-UnderstandingChannels](https://github.com/gophercon/2017-talks/blob/master/KavyaJoshi-UnderstandingChannels/Kavya%20Joshi%20-%20Understanding%20Channels.pdf)
- channelのソースコードを読む
    - https://golang.org/src/runtime/chan.go
- `channel`とは
    - goroutine間でデータをやり取りする仕組み
    - Queueによるデータ保存、やりとりするデータの型を持つ
    - Queueが満杯のときは処理をブロックし、`close(chan)`することで送信側の終了を受信側に伝えることも出来る
    - バッファあり/なしがある
        - channel生成時に容量を指定するとバッファありchannel。
- channelの実体は`hchan`構造体のポインタ。ヒープメモリに配置
    - https://golang.org/src/runtime/chan.go#L31

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```
- バッファありchannel(`hchan`構造体)の主な内部実装
    - データ保持用の循環リスト(配列)
    - データを格納する時、配列のどの位置に入れるべきかの送信用インデックス
    - データを取り出す時、配列どの位置から出すべきかの受信用インデックス
    - 受信待機キュー
    - 送信待機キュー
- channelのバッファを超える送信がされた時何が起きるか
- 1. 送信をブロック
    - スケジューラに対して止めるように指示(`gopark`)
        - https://golang.org/src/runtime/proc.go#L273
    - OS Threadは対象のgoroutineを止めた後実行から外して次のgoroutine
        - https://golang.org/src/runtime/runtime2.go#275
- 2. channelから見たブロック
    - goroutineの停止命令
    - 今のgoroutineと送信用の値を送信待機キューに保存(`sendq`内の`sudog`構造体に保存)
        - https://golang.org/src/runtime/runtime2.go#275
- 3. goroutineの再開
    - シンプルな取り出す処理
    - 送信待機キュー(`sudog`)に記録していたpendingされた送信用の値をQueue配列（`buf`）に追加
    - 送信待機キュー(`sudog`)に記録していたgoroutineを実行待ち状態にする
- channelの受信側がブロックする場合は何が起きるのか
- 1. 基本的には送信側のブロックの場合と逆
    - （受信待機キューで似たようなことが起きるイメージ？）
- 2. goroutineを再開する
    - 受信待機キュー(`recvq`内の`sudog`）にある要素にデータ保持用の循環リスト(`buf`)の内容を渡す
    - 受信待機キュー(`recvq`内の`sudog`)に記録していたgoroutineを実行待ち状態にする
- バッファなしの場合は常にブロックが発生。基本的にはバッファありと同じ
- `select`で使うとどうなるか
    - 全部のchennelがロック、待ち状態になり、勝ったcaseのところから再開される

なんとなく使っていたchannelが実際にどういう実装になっているか図解されながら丁寧に解説されていた。  
シュラスコを盛り込んだ挙動の例も妙な親しみ？とっつきやすさ？があってよかったと思う。  
発表手法としても見習いたい。  
ちゃんとGopherConの動画を見ながらスライドも読もう。


# LT1: vgo(Versioned Go Prototype) #golangtokyo
[@Asuka Suzuki](https://twitter.com/tanksuzuki)  
https://speakerdeck.com/tanksuzuki/vgo-versioned-go-prototype-number-golangtokyo

[全人類をgopher化する](https://gopherize.com/)tankszukiさんの`vgo`の発表

https://gopherize.com/

- vgoは[Russ Cox](https://swtch.com/~rsc/)が2018年2月に発表したgoコマンドに導入される予定のパッケージ管理機能
    - https://research.swtch.com/vgo
    - https://github.com/golang/vgo
    - https://github.com/golang/go/wiki/vgo
- go1.11でテクニカルレビュー、go1.12で導入予定
- vgoで変わること
- 1. ビルド時に依存性解決
    - `go get`なして`vgo build`
    - `go.mod`ファイルで依存性管理
    - `go.mod`ファイル内ではセマンティックバージョニングで管理
- 2. vendorディレクトリ不要
    - `$GOPATH/src/v/`配下でバージョンごとに管理
- 3. 破壊的変更が入る時はimport pathを変える(これは作成者の性善説？)
    - **後方互換義務化**
- 4. 可能な限り古いバージョンが選択される
    - 「可能な限り最新」はビルド再現性に問題がある
    - 新しいバージョンに向けたいときは`go.mod`を編集


`vgo`の発表は初めて聞いた。なんだかんだドキュメント読んでいなかったので、とても助かる。  
ほぼスライドの引用になってしまったのだが、図示などがあるのでぜひスライドで詳細を確認したほうが良いと思う。

ちなみにドキュメントの日本語訳もある

**和訳: Go & Versioning**  
https://qiita.com/nekketsuuu/items/36f00484ff7c30fd2007


# LT2: 自社のメインプロダクトにGoを導入したぞ＋＋
[@kurikazu](https://twitter.com/kurikazu)  
https://www.slideshare.net/kurikazu/go-94167920

スマホアプリのバックエンドサーバ間のAPIゲートウェイをGoでリプレースした話

- プロジェクトメンバーがみんなGo初心者の状態から始めた
    - A Tour of Goから始める
- 課題1:フレームワークの選定
    - 使わず1から書いた
    - 言語知識が浅い状態でコード自動生成などをしても辛いだけだった
- 課題2: 正しい書き方がわからない
    - メンバーみんなでコードを書いてレビューすることでチームレベルを上げていった
    - テストコードを書いていたので、理解と共にリファクタリングも出来た
- 課題3: 運用について
    - 仕様書はMarkdown、Swagger、Drone.ioでビルド、Jenkinsでデプロイ
    - ローリングアップデートの方法はわからなかったので、随時LBから切り離してバイナリを再起動している

フレームワークを使わなかったってことはswagger定義は自動生成しているわけではない？  
バイナリの再起動ということはコンテナ化まではしていない？  
新しい言語への挑戦に限らず、全員のドメイン知識が低い状態で新しいプロダクトに挑戦する時のチームビルディングとしても参考になるかもしれない。  
もっと詳しい話聞けたらなと思った。

# LT3: linterを作ってみよう
[@daisuzu](https://twitter.com/dice_zu)  
https://daisuzu.github.io/golang-tokyo-14/#1

linterを作ってみた話。

https://github.com/daisuzu/gsc

現段階では`context.Context`のスコープをチェックする機能が実装済み。

- チーム独自ルールなども静的解析したいと思った時、既存のツールでは要求を満たせない
- いくつかのパッケージを使えば静的解析ツールを作ることができる
    - [go/ast](https://golang.org/pkg/go/ast/)
    - [go/parser](https://golang.org/pkg/go/parser/)
    - [go/token](https://golang.org/pkg/go/token/)
    - [go/types](https://golang.org/pkg/go/types/)
    - [golang.org/x/tools/go](https://godoc.org/golang.org/x/tools/go)
-  とはいえlinterを作るのは面倒事が多い。
    -  ファイル、ディレクトリ、パッケージで異なるASTの取得処理
    -  フォーマットやexit codeの処理
    -  ASTが絡むテストの用意
-  `honnef.co/go/tools/lint`を使うとラク(GitHubのURLはちょっと違う)
    -  https://godoc.org/honnef.co/go/tools/lint
    -  実装、テストが（何も使わないよりは？）ラクに書ける

`honnef.co/go/tools/lint`初めて知った！パーサーやろうやろうとしていて全然できていないので、静的解析ツールの作成から入ると実用目的でモチベーション保ちながらできるかな？

# 関連
- [[Go]golang.tokyo #13 - Macでハンズオンアプリを動かすまでに必要だったこと #golangtokyo](/2018/03/14/report-golang-tokyo-13-android-with-gomobile/)
- [[Go]golang.tokyo #12 参加メモ #golangtokyo](/2018/01/31/golangtokyo-12/)
- [[Go]golang.tokyo #11 参加メモ #golangtokyo](/2017/12/12/golangtokyo-11/)





