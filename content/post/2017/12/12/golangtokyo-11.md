+++
title= "[Go]golang.tokyo #11 参加メモ #golangtokyo"
date= 2017-12-12T09:39:32+09:00
draft = false
slug = ""
categories = ["report","go"]
tags = ["golang","golangtokyo"]
author = "budougumi0617"
+++

|||
|---|---|
|URL|https://golangtokyo.connpass.com/event/72986/|
|会場|TECH PLAY SHIBUYA|
|日時|2017/12/11(月) 19:00 〜 21:50|


# 闇のBashをGoに置き換える技術(nashiox)

- 怖くて触れないけどなぜか動いているBashスクリプト(10ファイル2128行)をGoに置き換えた。
- データ加工、ダンプ、インポートを行っていたスクリプト。GNU Parallelも使っていたりしたが、そのへんはgroutineで。
- Goならば`os/exec`によるスクリプト実行やbashライクの書き方を許容しながら少しずつ移管できる。

たしかに[`os/exec`](https://golang.org/pkg/os/exec/)で絡めて段階的にアプローチしていくの良さそう。

# How Go test tests(timakin)

https://speakerdeck.com/timakin/how-go-test-tests


- 関数がどんな出力をするのかをgo1.9から[Examples](https://golang.org/pkg/testing/#hdr-Examples)で書けるようになった。

- [`src/testing`](https://golang.org/pkg/testing/)にContributeしようとしたけど、テストの実行結果ってどうテストするんだ？
- [`src/cmd/go/go_test.go`](https://github.com/golang/go/blob/master/src/cmd/go/go_test.go)にgo commandsのテストが書いてある。
- テスト用のFixtureは[`src/cmd/go/testdata/src`](https://github.com/golang/go/tree/master/src/cmd/go/testdata/src)の下にある。[`all.bash`](https://github.com/golang/go/blob/master/src/all.bash)で動けばだいじょぶ！

関数出力までExamplesに書けるようになってたの知らなかった。  
標準パッケージの構成知らないこといっぱいあって読めば学び多いんだろうな、そして読まないとなと思った。



# 5分で学ぶGo2(niconegoto)
- Go2はまだ`Proposal-Accepted`になったProposalがない。OpenなIssueも賛否両論ばかり。
	- https://github.com/golang/go/issues?q=label%3AProposal-Accepted+label%3AGo2+is%3Aclosed
- Rob Pikeが書いたProposalは賛同の意見が多いので、Go2で実現しそうな有力仕様？
	- https://github.com/golang/go/issues?utf8=%E2%9C%93&q=is%3Aopen+author%3Arobpike+label%3AProposal+label%3AGo2
- [`Proposal`, `Go2`](https://github.com/golang/go/issues?utf8=%E2%9C%93&q=is%3Aopen+label%3AProposal+label%3AGo2)でIssue注視していると面白いかも

golang/goのIssueや日々のコミットを追っていくの少しずつだけど自分も始めたいなと思った。

# go-sql-driver/mysqlの問題に救われて、そして更新したら死んだ話(shinofara)

- `go-sql-driver/mysql`を使うとき、[`SetConnMaxLifetime`](https://golang.org/pkg/database/sql/#DB.SetConnMaxLifetime)をセットしていなかったので、（振り返ってみれば）コネクションがすぐ切れていた。
- 今までは再接続されていたのでなんとかなっていた。危ないのでPRで修正されて再接続されなくなった。
- 「Goならよしなにしてくれる」なんて思わずにやることはちゃんとやっておかないと危ないよという教訓。

Goじゃなくとも便利なパッケージや言語仕様に頼っていると、たしかに思わぬところで足を救われる…  
全部は無理だが普段使っている機能がどうやって動いているのか知るのは大事だと思った。（自分はGUI作る時に下調べが甘くて後半で設計の修正が必要になったりする）


# DIコンテナを使わないDI(morikuni)
https://speakerdeck.com/morikuni/golang-dot-tokyo-number-11

- 依存関係を解決する３つのアプローチ
- 1. `main`にかく
	- オブジェクトが増えるたびにmainが肥大化してしまう
- 2. DIコンテナを使う。たとえば[goldi](https://github.com/fgrosse/goldi )を使う。
	- DIコンテナのDSL覚えないといけない。
- 3. DI用のInject関数を定義する。あるオブジェクトについて依存先が引数0個で取得できる関数。
	- 依存先が増減してもInject関数内に影響が納まる。
- 詳細は2017/12/12の[Go4のアドベントカレンダー](https://qiita.com/advent-calendar/2017/go4)で。
	- [GoにおけるDI](http://inukirom.hatenablog.com/entry/di-in-go)

ファクトリメソッドパターンとか、ビルダーパターンの`Director`的なのを作るってことかな？  
Plugin使えばGoでももっと動的なDIができるのだろうか(C#だとDLLの入れ替えとかでビルド後でもDIコンテナを変更できる）

# Go開発を助けるGoツールGo選のGo紹介(haya14busa)
https://docs.google.com/presentation/d/13DQUhFkmAf25ItA1Gpuw0LApEnqhQB-qKN1D-SX7RWk/edit

- goplay コマンドライン用のThe Go Playground client
	 - https://github.com/haya14busa/goplay
- gopkgs めちゃくちゃ速くローカルのパッケージ一覧を出す。pecoなどとも相性がよい。
	- https://github.com/haya14busa/gopkgs
- goverage 複数パッケージのカバープロファイルがとれる。ただしGo1.10から標準で出来るようになる。
	- https://github.com/haya14busa/goverage
- reviewdog 差分結果のLintだけ出してくれる。
	- https://github.com/haya14busa/reviewdog
- gofmtrlx ちょっとしたカンマ忘れとかを自動修正してフォーマットしてくれるツール
	- https://github.com/rhysd/gofmtrlx


カンマ忘れとか良くするので、`gofmtrlx`よさそう。reviewdog使おうと思って使えてないから、今年中に自分のリポジトリに設定しておきたい。

# GoでJavaのライブラリを使う(juntaki)
https://speakerdeck.com/juntaki/gokarajavafalseraiburariwoshi-u-number-golangtokyo


- 「ScalaやKotlinをJVMの既存資産を使えるって？それGoでも出来るよ！」って言いたくて`jnigo`と`javago`を作った。
- cgo経由でJNIを使ってGoからJavaのライブラリを使う。
- jnigo JNI wrapper for Go
	- https://github.com/juntaki/jnigo
- javago Java wrapper code generator for Go.
	- https://github.com/juntaki/javago

発想とそれを実現させてしまう :muscle: 腕力 :muscle: 見習いたい！

# go tool traceを使ってみよう(nametake)
- `pprof`以外にも`go tool trace`も使ってみよう
- ヒープ領域、goroutineの数とか、プロセスの状態とかのタイムラインをweb Viewで見れる。
- System Callとかの呼び出しとか、GCの状況も確認できる。

普通に`fmt.Fprintf`していたときと`bufio`でバッファ使ってみたときでsyscallの数がどう変わったか、などサンプルベースで説明されたいたのでわかりやすかった。

他の方の記事だが、参考記事。  
[go tool traceでgoroutineの実行状況を可視化する](http://yuroyoro.hatenablog.com/entry/2017/12/11/192341)


# Go1.10導入予定のキャッシュビルドについて(zchee)
- Go devel, Go1.10beta1でキャッシュビルドが使えるようになった。
- `GOCACHE`環境変数が増えている。キャッシュの保存場所。
- キャッシュはsha256でハッシュ化されて保存される。雰囲気は`ccache`に近い感じ。
- devl使っていきましょう！

業務で使っていない自分こそdevelopガンガン使える立場だ。（自分はちょっと使って何かしらのツールが動かなくなって辞めるの繰り返しをしている）  
発表にはなかったけど、Go1.10ではテスト結果もキャッシュされるようになってコードに変更がないとテストが実行されなくなるとか  
（無理やり実行するには`go clean -testcache`しないといけない）  
https://github.com/golang/go/blob/496688b3cf81f92ee55c359cfbd8cd2c9d71c813/src/cmd/go/alldocs.go#L215-L218

```go
// The -cache flag causes clean to remove the entire go build cache.
//
// The -testcache flag causes clean to expire all test results in the
// go build cache.
```

# 今年作ったGoツール10(?)選(Songmu)

- horenso cronラッパー
	- https://github.com/Songmu/horenso
- go-httpdate Perlのhttpdateの移植
	- https://github.com/Songmu/go-httpdate
- ghg GitHub Releasesのインストーラ
	- https://github.com/Songmu/ghg
- paranoidhttp プライベートネットワークへのアクセスを禁止する
	- https://github.com/Songmu/paranoidhttp
- go-memcached-tool Perlのmemcached-toolの移植
	- https://github.com/Songmu/go-memcached-tool
- blogsync はてなBlog用のCLIクライアント
	- https://github.com/motemen/blogsync
- ghch Gitのもろもろからチェンジログが作れる
	- https://github.com/Songmu/ghch
- posttailer ログ監視している時にログローテートが起きてもよしなにしてくれる。
	- https://github.com/Songmu/postailer



多すぎて5分では全部紹介しきれていなかった。業務で書く→OSSで公開する。こういうサイクルってエンジニアとしてのスキルアップに重要なんだろうなと常々思っている。


# Machine Learning with Go を読んで(ysaito)
https://speakerdeck.com/ysaito8015/machine-learning-with-gowodu-nde

- Machine Learning With Go
	- https://www.amazon.co.jp/dp/B01LPRN11G/
- ブログに書いてある部分だけでも試してみるのおすすめ
	- http://www.datadan.io/building-a-neural-net-from-scratch-in-go/

英語のMeetupも主催されているとのこと。
https://www.meetup.com/ja-JP/Tokyo-Gopher-Hub/

機械学習、教養レベルだけでも覚えておくべきなんだろうけどなかなか手が出せていないのでこういう自分の好きなことからつながっている入り口があるのは嬉しい。


# MP3 デコーダを C から移植した話(星一)
https://docs.google.com/presentation/d/e/2PACX-1vTTXf-LWNRvMVGQ7GI4Wh8EKohot_9CMtlF4dswpYGpuYKOek5NeNP-_QZnNcRFZp9Cwm0pCcykjqDN/pub?start=false&loop=false&delayms=3000&slide=id.p


- go-mp3 `io.Reader`が使える`C`から移植したMP3 Decoder
	- https://github.com/hajimehoshi/go-mp3
- Oto A low-level library to play sound on multiple platforms
	- https://github.com/hajimehoshi/oto
- `io.Reader`と`io.Writer`のAPI用意したので`io.Coopy`でMP3データがコピーできる。
- ひたすらCの関数ひとつひとつずつGoに書き直した。
- テストは耳でテストした。デグレしたら`git bisect`


五感でテストというパワーワードがタイムラインに舞っていた。  
git bisectは初めて知ったので、後で調べる。別の方の記事だが参考  
[git bisect で問題箇所を特定する](https://qiita.com/usamik26/items/cce867b3b139ea5568a6)

# 最強のDatastoreライブラリを作った(vvakame)
https://speakerdeck.com/vvakame/golang-dot-tokyo-number-11

- Datastore。Googleのfull managed scalable DB
	- https://cloud.google.com/datastore/?hl=ja
	- AppEngine Datastore, Cloud DatastoreのAPIが異なってたり、
	- `context`もらえないことがあったり、
	- `struct`に特定の型しか使えなかったり、
- なので作った
	- https://github.com/mercari/datastore
	- https://qiita.com/vvakame/items/9310bcb5a4e87888d505
- 技術書展の公式Webのバックエンドで動いている。サークル一覧APIとかRPCめっちゃ少なくなった。
	- https://techbookfest.org/
- 事例がほしい。使い方とかはgcpugのSlack(`#g-cloud-datastore_ja`)とかで問い合わせてくれれば教える！
	- http://gcpug.jp/join

まずDatastore(GCP)から勉強しないといけないなーと思った。やっぱりGoやってるしGAE使ってみないとだ。

# Goで地図サーバを構築した話(tsukumaru)

- OpenMapTilesが元データ。地図のタイルデータを入手できる
	- https://openmaptiles.org/
- go-staticmaps。タイルデータにマーカーなどを立てたあとの画像を作れる。
	https://github.com/flopp/go-staticmaps
- go-staticmapsはコマンドラインからも使えて便利。

出力画像とかはご本人のqiita記事みるとよい。  
[OpenMapTilesとgo-staticmaps](https://qiita.com/tsukumaru/items/62b54a45ece1d47c9d1a)

# Alfred Workflows by Go（仮）(enta)
https://speakerdeck.com/endotakuya/alfred-workflows-by-go

- MacのAlfred。workflowを自作できる。
- Goで作れる。
- 既存のライブラリが微妙だったので自作した。
	- https://github.com/endotakuya/alfred-go


Alfredでアプリ起動しかしていないので自作ワークフローが作れたり、Sleep Displayとか出来るの初めて知った。。。！


# 感想
LTオンリーだったが濃い内容ばかりだった。  
自分は`Go`少しかじっているだけで「開発」に使ったことがないので、自分でサービスやツールを作ってみて「何か」と連携させたり適用させてみないとスキルアップできないなーと思った。  
来年の課題である。いや来年と言わず今すぐやるべき課題である。年末に参加できてよかった。


