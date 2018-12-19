+++
title = "golang.tokyo #20 忘年LT大会！！ 参加メモ #golangtokyo"
date = 2018-12-19T11:12:16+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["report", "go"]
tags = ["golang"]
+++

golang.tokyo #20に参加してきたのでメモ。

<!--more-->

|イベント名|golang.tokyo #20 忘年LT大会！！|
|---|---|
|URL|https://golangtokyo.connpass.com/event/111077/|
|会場|株式会社メルカリ 東京都港区六本木6-10-1(六本木ヒルズ森タワー18F)|
|日時|2018/12/18(火) 19:30 〜 22:00|
|ハッシュタグ|[#golangtokyo](https://twitter.com/hashtag/golangtokyo)|

# Dive panic & type
[@takochuu](https://twitter.com/takochuu)

- https://speakerdeck.com/takochuu/dive-panic-and-type
- `runtime`pkgのの中のコードを見るとGoの実行モデルがわかる
- 例えば`panic()`の中身とか

当日は下で開始ぎりぎりまで受付のお手伝いをしていたのでほとんど聞けず。

# labelerrとsuberrを作った話
[@nametake](https://github.com/nametake)

- https://speakerdeck.com/nametake/created-labelerr-and-suberr
- `pkg/errors`だけではドメイン固有のエラーを作るちょっと良くない感じになるのでerrorのラッパーを作った
  - https://github.com/nametake/labelerr
  - https://github.com/nametake/suberr
- `Wrap`、`Add`で`error`を作って`SubError`を取り出したり`Cause`使ったりできる。

Go2で何かしら変わるようだが、Goのerror周りは各々で思考錯誤している感じだ（pkg/errorsを使うのはぼぼデファクトだが）。
エラーハンドリングはリリース後の運用フローにも強く影響を与えるところなのでしっかり目的意識を持って設計したいところ。


# パッケージ外から非公開フィールドを変更する方法
[@makki_d](https://twitter.com/makki_d)

- https://speakerdeck.com/makki_d/patukeziwai-karafei-gong-kai-huirudowobian-geng-surufang-fa
- 通常unexportedなフィールドはpkg外からアクセスできないが。。。
- リフレクションでゴリゴリするとできる
- ただし`reflect`だけでは非公開フィールドの変更はできない（参照まではできる）
- `reflect`と`unsafe`を組み合わせるとできる
  - `unsafe.Pointer(v.UnsafeAddr())`

興味本位でリフレクション使ってunexportな値まで取得したことはあったが、書き込みまでしたことはなかったので参考になった。

# サービス開発に役立つfmtテクニック
by imoty

- verbsの`%#v`や`%+v`は便利
- 引数インデックス(ex: `%[2]d`)なども使える
  - https://golang.org/pkg/fmt/#hdr-Printing
- `Formatter`や`Stringer`のインターフェースを適切に実装して個人情報のマスキングとかする
  - https://golang.org/pkg/fmt/#Formatter
  - https://golang.org/pkg/fmt/#Stringer

引数インデックスはあまり使ったことがなかった。fmtだけでも奥が深い…

# メジャーな Live Reloaderの違いをちゃんと調べて見た
[@yudppp](https://twitter.com/yudppp)

- https://speakerdeck.com/yudppp/compare-major-live-reloader-of-go
- 開発中の.goファイルの変更に対してホットリロードをしてくれるライブラリはメジャーどころで３つ
  - https://github.com/codegangsta/gin
  - https://github.com/gravityblast/fresh
  - https://github.com/oxequa/realize
- realize以外はあまり活発に更新されていない
- Docker内でホットリロードしたいときはrealize

freshしか知らなかったのでrealize使ってみようと思った。複数プロジェクトのコードも監視してくれるらしくて便利そうだ。

# Goでテスト仕様書からJSONファイルを自動生成するツールを作ったよ
[@takuokl](https://twitter.com/takuokl)

- https://speakerdeck.com/takuoki/the-json-generator-tool-based-on-google-spreadsheets
- Goはユニットテストについての言及はいろいろあるが、インテグレーションテストについてあまり資料が無い。
  - [直近ではGoConで話していたtimakinさんくらい](https://speakerdeck.com/timakin/golang-api-testing-the-hard-way)
- テスト定義書からテストデータのJSONファイルを出力するOSSを作った
  - https://github.com/takuoki/testmtx
  - Google SpreadsheetsからJSONファイルを出力する
- ブログ記事もある。
  - テスト定義書からテスト用JSONファイルを生成するツールをつくってみたよ
    - https://medium.com/veltra-engineering/a-tool-to-generate-test-json-files-from-a-test-definition-sheets-c36b77ab886f

Google Spreadsheetsでテストパラメータを管理できるのはQAの人とやり取りしやすそうな気がする。
便利だ。

# Goでデスクトップアプリを作る
[@micchiebear](https://twitter.com/micchiebear)

- https://speakerdeck.com/mishirakawa/go-astilectron-godedesukutotupuapuriwozuo-tutemitakunatutatoki
- WinでもMacでも動くデスクトップアプリをGoで作りたい
- Electronみたいなことをできるgo-astilectronというライブラリがある。
  - https://github.com/asticode/go-astilectron
- bundlerとかbootstrapなどもある
  - https://github.com/asticode/go-astilectron-bundler
  - https://github.com/asticode/go-astilectron-bootstrap
- フロント側のHTMLやJSを作ってバックエンドをGoで実装できる

まさかデスクトップアプリも作れるとは。
[go-mobile](https://github.com/golang/mobile)もあるしGoは本当にマルチプラットフォームだ！！（簡単にできるとは言っていない）


# いまさらdatabase/sql
[@rock619](https://github.com/rock619)

- `sql.DB`は毎回`Close()`する必要もない。`Ping()`などを適切に使う
  - https://golang.org/pkg/database/sql/#DB
- 定義済みエラーのハンドリングも忘れずに
  - https://golang.org/pkg/database/sql/#pkg-variables
- `NullString`などを適切に使う
  - https://golang.org/pkg/database/sql/#NullString
- `sql.DBStats` structにはGo1.11から多数フィールドが追加されてより便利になった。
  - https://golang.org/pkg/database/sql/#DBStats


Go1.11からいろいろ増えているのは知らなかった。
私はいつも他の人が作った内製ラッパーでDB叩いているだけでこの辺ちゃんと理解できていない。一度ちゃんとパッケージ読んだほうが良さそう。


# 君の並行処理は実行するまでもなく間違っている
[@y_taka_23](https://twitter.com/y_taka_23)

- https://speakerdeck.com/ytaka23/golang-dot-tokyo-20th
- こちらの論文の内容の紹介
  - A Static Verification Framework for Message Passing in Go using Behavioural Types
    - http://mrg.doc.ic.ac.uk/publications/a-static-verification-framework-for-message-passing-in-go-using-behavioural-types/
- 並行処理はテストしずらい。だいたいのところがE2Eテストと運用に乗せたあとに検知するほうに寄せている
- 振る舞い型による抽象化を使えば実行しないでも実装ミスを検知できる

正直私のレベルでは理解できなかった。
チャネル周りの実装を静的解析みたいなことをして正しく実装されているか（振る舞えるか）を検知できる手法らしい。

実装はこれなので試してみる。

- http://mrg.doc.ic.ac.uk/tools/#godel-checker
- godel-checker
  - https://bitbucket.org/MobilityReadingGroup/godel-checker

# vim-goの便利機能
[@gorilla0513](https://twitter.com/gorilla0513)

- https://docs.google.com/presentation/d/1RXjJM7KUPkeFaUgoW_mA2bv4ms9ZiLLUuuMFIH12-J4/edit
- vim-goでいろいろできるGoの便利機能
  - GoAddTags, GoFIllStruct etc...
- qiitaにもまとめを書いている
  - https://qiita.com/gorilla0513/items/a027885d03af0d6d5863
- dockerのCUIツールを作った。
  - https://github.com/skanehira/docui

VimでGo書くときのTipsを実行例付きで紹介されていた。
VSCodeも似たようなことができるので、自分のエディタで出来ることを一度調べておくといいかもしれない。

- https://github.com/Microsoft/vscode-go#commands

ちなみにvim-goを使えばデバッグもできる。

- [[Go]vim-goとDelveでVim上からGoのデバッグを行う #vim #golang](/2018/05/30/debug-go-by-vim-go-and-delve/)

# 業務でよく使っているライブラリ&ツール紹介
[@qushot](https://twitter.com/qushot)

- https://github.com/labstack/echo
  - 言わずと知れたWebフレームワーク
  - `HideBanner`オプションを使うと起動時のロゴ表示をオフにできるらしい。
- https://github.com/google/uuid
  - UUID生成
  - ユニークな文字列ほしいときも使える
- https://github.com/go-playground/validator
  - バリデータ
  - struct fieldにvalidate tagをつけておくと検証ができるらしい
- https://github.com/swaggo/swag
  - GoDocからSwagger形式を生成できる
- https://github.com/swaggo/echo-swagger
  - echoの定義からSawgger形式を生成できる。
  - gin-swaggerなどもある

バリデータは知らなかった。便利そう。

# Bounds Check Eliminationについて調べてみた
[@kaznishi1246](https://twitter.com/kaznishi1246)

- https://speakerdeck.com/kaznishi/1218-lt
- Bounds Check Elimination(BCE)はGo1.7から追加された機能
- コンパイラが安全だと判断する状況下で範囲チェックの省略ができる
- 確認したいときはビルド時に`-gcflags="-d=ssa/cechk_bce/debug=1"`とつけてビルドする
- 条件分岐で確実にlenの範囲内になるときとか、forループとかでも適用される。コンパイラ賢い。

GoConでも発表されている方がいたが、BCEという機能について全然知らなかった。
みなさんリリースノートなどをちゃんと読んでいるから知っているのかな？実装時に意識していきたい。

# Goでのモデル 取扱説明書
[@uqichi](https://twitter.com/uqichi)

- https://speakerdeck.com/uqichi/golang-how-to-deal-with-models
- DTOらへんのモデルの話。
- ModeだけでなくMarshal/Unmarshal定義したりEntityをつくったりするけれど全部モデルだけで済ませたい
  - ORM。Active Recordパターン
- https://github.com/gobuffalo/pop
  - マイグレーションできる
  - アソシエーションに対してEager Lodingなどもできる


[gobaffalo](https://gobuffalo.io/en)は知っていたけどそんなライブラリがあったのは知らなかった。gobuffalo/popはマイグレーションもできるらしいので、あとで機能ちゃんと読む。

# アーキテクチャについて思っていること
[@sonatard](https://twitter.com/sonatard)

- https://speakerdeck.com/sonatard/akitekutiyanituitesi-tuteirukoto
- 依存していないとは？インポートしていないことなのか？直接関数を実行していないことなのか？
- レイヤー図の矢印は何を示しているのか？（示していると認識しているのか？）
- パッケージ分割 != アーキテクチャの決定
- フレームワークの実装や規格に思考を引っ張られてしまうのはいけない。
- 実装をアーキテクチャと勘違いしないこと
- ３つを考え抜く
  - 思想を正しく理解すること
  - レイヤーがどのような責務について関心しているのかを定義すること
  - どのように実装するか

今新規設計を始める前の段階でこの辺モヤモヤしていたので非常に刺さった。
年末年始考え抜いて言語化しよう。

# Manages tool dependencies
[@izumin5210](https://twitter.com/izumin5210)

- https://speakerdeck.com/izumin5210/how-to-manage-tool-dependencies-in-go
- 依存パッケージはバージョン管理できる。が、ツールのバージョン管理どうする？
  - `golint`や`mockgen`みたいなコードジェネレータ周りはチームで統一されていないと挙動が変わる
- tools.goでやるのが今のところベターっぽい
  - https://github.com/golang/go/issues/25922#issuecomment-412992431
- ただ毎回ビルドし直したりするのは辛い
- （いずれは解決されるだろうが、）gex作った。
  - https://github.com/izumin5210/gex
- Wantedlyでは毎月k8s読書会もしている
  - https://www.wantedly.com/projects/268180

この辺は最近も`protoc-gen-go`あたりでハマったりしてたので同じ気持ちになった（バージョン管理している`protobuf`のimportとCI上で`go get`した最新のprotoc-gen-goが生成するコードが合わなくてCIがフェイルするようになった）。
困るだけじゃなくて自分で解決方法を作れるのが素晴らしい。見習いたい。

# GoでDialogsを使ったSlack Appを作る
[@shiimaxx](https://twitter.com/shiimaxx)

- GoでDialogsを使ったSlack Appを作る
  - https://shiimaxx.hatenablog.com/entry/go-dialogs-slack-app
- GoでSlack Appを作るときにフォームを表示してユーザーの入力をインタラクティブに扱うことができる
- https://github.com/tcnksm/go-slack-interactive
- GolangでSlack Interactive Messageを使ったBotを書く
  - https://tech.mercari.com/entry/2017/05/23/095500


職場でもSlack BotをGoで作っていたりするが、Slackメッセージ的には単純な応答やリマインダをするだけなので機会があれば導入してみたい。

# Exportされてないフィールドを書き換えるなよ！絶対だぞ！絶対！
[@inukirom](https://twitter.com/inukirom)

- https://speakerdeck.com/morikuni/golang-dot-tokyo-number-20
- `unsafe.Pointer`経由でexportされていない値を変更する
- https://github.com/morikuni/go-experiment/tree/master/overwrite

発表順に負けてしまったが、力技でunexportな値を変更する話その２。
ライブラリのテストでどうしてもやりたくなるときはたしかにある。

# Wire: コード生成によるDI
[@_ishkawa](https://twitter.com/_ishkawa)

- https://speakerdeck.com/ishkawa/introducing-wire-dependency-injection-by-code-generator
- https://github.com/google/wire
  - Googleが作ったDependency Injection(DI)ツール
- DIのコードで本質的な部分はどこだろう？
  - どのような依存があるのか
  - どうやって依存を取得するのか
- 本質以外のコードを自動生成するのがWire
- コンパイル時に依存関係のグラフの欠損も確認できる

非常にわかりやすかった。自分もWireについて発表したりしていたので、なおさら、必要性とWireを使って享受できるメリットがシンプルにまとまっているプレゼンだったなと思った。
ブログにもっと具体的なサンプルコードも書かれている。

- Wire: Goのコード生成によるDIツール
  - https://blog.ishkawa.org/2018/12/10/1544371624/

（私もちょっと書いてはいる（宣伝））

- [[発表資料]go-cloudとWireを利用したDI #gounco #go](/2018/10/19/presentation-gounco-lt4/)
- [google/wireを使ったDIとDI関数のシグネチャについて #go](/2018/12/14/how-to-use-google-wire/)

# レイトレーシングと Goroutine by sachaos
[@sachaos](https://twitter.com/sachaos)

- https://speakerdeck.com/sachaos/reitoresingutogoroutine
- https://github.com/sachaos/go-simple-raytracer
- レイトレーシングをフルスクラッチをしてみた
- 標準pkgだけでごりごり書ける
  - vector周りは腕力で実装した
- GPUで並列計算するところをgoroutineで並行処理をしている
- pprofでみるとgoroutine safeな`rand/math`のロック周りで速度がでなかったので別ライブラリを使った
  - https://github.com/mono0x/prand

あまりGoでグラフィック系の実装をしたことがなかったので面白そうだと感じた。


# NodeJS → Go
[@caust1c](https://twitter.com/caust1c)

- https://slides.com/abraithwaite/js-go#/
- Node.JsからGoへアーキテクチャのマイグレーションを行なった
- マイグレーションにあたり、Node.jsとGoを同時稼働してリアルデータで検証をした
- 型なし言語から型あり言語へのマイグレーションには乗り越えなければいけない壁がある

Alanさんはたまたま日本に来日していて、「Goのイベントがあるなら参加して良い？」と遊びにきてくれた方。Gopherはパッションがすごい！
Twitterみたら昔読んですごい参考にした記事を書いた方だった。

- 5 advanced testing techniques in Go
  - https://segment.com/blog/5-advanced-testing-techniques-in-go/


# 所感
20本の様々なLTを聞くことが出来てとても勉強になった。知らないライブラリや標準パッケージの中身も知ることが出来た。
昨日の今日でしっかり読み解けてないものが多いので引き続き初耳のライブラリなどは確認していく。
あとNature Remoが欲しくなった。

- https://github.com/tenntenn/natureremo

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">LTの切替くんはNature Remo APIをGoから叩いてます <a href="https://twitter.com/hashtag/golangtokyo?src=hash&amp;ref_src=twsrc%5Etfw">#golangtokyo</a></p>&mdash; tenntennʕ ◔ϖ◔ʔ ==Go＠Goが生きてる (@tenntenn) <a href="https://twitter.com/tenntenn/status/1074988877289377792?ref_src=twsrc%5Etfw">2018年12月18日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



# 関連
- [golang.tokyo #14 ゴールーチンとチャネルによる並行処理 参加メモ #golangtokyo](/2018/04/19/golangtokyo-14-goroutine/)
- [[Go]golang.tokyo #13 - Macでハンズオンアプリを動かすまでに必要だったこと #golangtokyo](/2018/03/14/report-golang-tokyo-13-android-with-gomobile/)
- [[Go]golang.tokyo #12 参加メモ #golangtokyo](/2018/01/31/golangtokyo-12/)
- [[Go]golang.tokyo #11 参加メモ #golangtokyo](/2017/12/12/golangtokyo-11/)

