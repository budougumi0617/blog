+++
title= "Go Conference 2017 Autumn参加メモ #gocon"
date= 2017-11-09T01:27:48+09:00
draft = false
slug = ""
categories = ["Go"]
tags = ["golang"]
author = "budougumi0617"
+++




|||
|---|---|
|URL|https://gocon.connpass.com/event/66615/|
|会場|株式会社メルカリ|
|日時|2017/11/05(日) 09:00 〜 17:30|
|toggeter| https://togetter.com/li/1168291 |


## 感想

自分は[gopl](http://www.gopl.io/)の練習問題や簡単なツールしかGoで作ったことがないので、サービス・アプリケーション構築におけるGoのプラクティスや利用OSSの知見を聞くことが出来て非常に良かった。聞いたOSS、サービスのリソースはだいたい探せたので、復習がてらひとつずつ確認しておきたい。
また、多くの発表者の方がスライドを英語で書かれており、英語弱者の自分としては必要なのは「プログラミング言語」だけではないなと痛感した。
贅沢にもおみやげにGopher Tシャツまで頂いた。Googleさん、Mercariさん、運営のみなさま、Gopherのみなさまありがとうございました。


# 特別企画：パネルディスカッション w/ mattn

mattnさんやkaoriyaさんら4名の公開ディスカッション。
質問内容は事前に`gophers.slack.com`などで募集されていた。いくつかはその場での質問もあり。

印象に残った質疑応答

Q. 仕事や趣味でどうGoを使ってますか。

- 公共事業/サーバーエンド/IoTデバイス/アルゴリズムのプロトタイプ etc...

Q. Golangの運用で苦労したことありますか？

- 印象に残るような苦労はないとのこと。
- シングルバイナリなのでアップグレードのときにあまり苦労しない。
- 依存関係が少ないのでJavaとかRubyのときより苦労しなくなった。 

Q. WindowsでやるGoについて。

- 他の言語はWindowsのもろもろをUNIX側に合わせるような実装をしていることが多い。が、GoはちゃんとWindowsの仕様に則った実装を入れている。(`path/filepath`とか）
- Windowsで簡単にexeファイルをつくれるのは、非エンジニアを含めて内製ツールを配布する立場だとすごいラク。
- クロスコンパイルでもあまり困ったことがない。

Q. 書籍について

- Rob Pikeは体系的に最新の情報を得るために書籍は向いていないと考えているらしい。なので[The Go Blog](https://blog.golang.org/)などをよく更新して、Docに反映している。

Q. Go2について。

- 趣味でアルゴリズムかいていると`Generics`がほしいと思うことは正直ある。
- `go generate`らへんはもうすこしワンプッシュほしい。
- Goは安全に倒しすぎているところがある。ほぼ起きるはずないレースコンディション対応などで遅くなっているところもあるので、それを外す、あるいは外せるオプションがあればよい。

Q. 中級以上のGopherを採用するならば、どんな質問をするか

- `goroutine`と`channel`をちゃんと使えるが中級レベル。
- Workerをサクッと作れるか
- 起動数制限付きのワーカースレッドを作れるか
- `sync.WaitGroup`をちゃんと使えるか
- 珍しいパッケージ(`ssl/crypt`パッケージ）を読ませてちゃんと理解できるか。


Q. モックを階層構造をもっているテストはどうするのか

- モックなどはリフレクションとか使うしかないのかなと思う。自作することが多い。
- 正直`Assert`とか使いたいので`testfy`を使っている。  https://github.com/stretchr/testify

Q. GoとRDBMSのやりとり。

- `db/sql`で書くドライバは生の部分が直接見えないようになってる。リフレクションを多用しているので正直遅いところはある。
- ただ、知見は溜まってきているのでGo2でドラスティックに変更してもよいのではないかと思っている。
- テーブルファーストでコード生成するようなライブラリもあるので、そのGoライクっぽい。

Q. Contributionについて

- タイポの修正とかドキュメントの修正ならばすぐできそう。
- レビューの視点なども含めてかなり勉強になるのでぜひコミットしたほうがよい。
- Issueでも凄腕エンジニアとディスカッションすることが出来るので、オススメ

https://golang.org/doc/contribute.html

---

たしかに`channel`周りは他言語にない仕組みだし、ConncurrencyなGoプログラミングが出来れば中級者名乗れそうだと思った。コントリビューションについてはコミットメッセージの内容からがっつり見ていただけるとのこと。Gerrit登録しただけで終わっているので少しずつ流し読みから始めてみたい



# キーノート：Go at DigitalOcean
発表者：fatih

https://speakerdeck.com/farslan/go-at-digitalocean

https://blog.digitalocean.com/cthulhu-organizing-go-code-in-a-scalable-repo/

- DigitalOceanはRuby, Python, perl, JS, C++などを利用しているが、インフラはGoに移行している。
- 最初にGoを使ったのは2014年から。公式の依存性管理ツールなどがない時代から。依存性管理などの移行は随時行っている。
- MonoRepo構成でやってるが環境構築などはラク。`direnv`などを利用した`.env.sh`ですぐに開発が始められるように整えている。メンテナンス製に難は確かにある。
- CIは変更パッケージだけテストする`gta`などの内製ツールを利用して高速化を図っている。
- PRのマージ後は自動で内部向けサーバにバイナリがデプロイされる（Go+Monorepoだから簡単）
- プラットホームとして`Kubernetes`ベースの`DOCC`(Digital Ocean Control Center)を有している。
- deploy周りは内製CLIツール(`artifactctl`)で整備されている。

---

正直英語の発表は全然理解が追いつけないので、自分は半分も理解できていないと思う。
その他スライド中に名前が出たOSSは以下。
https://github.com/drone/drone
http://concourse.ci/
https://www.gocd.org/
https://github.com/GitHubJanitor/GitHubJanitor

# スポンサーセッション：Go + microservices at Mercari

発表者：deeeet

https://talks.godoc.org/github.com/tcnksm/talks/2017/11/gocon2017/gocon2017.slide#1


- メルカリはPHPのモノリシックなサービスだった。
- オーナーシップが不明確になってきたり、ベロシティが低くなってきたのでマイクロサービス化がUSで始まった。
- USのマイクロサービスは`API gateway`パターンを使っている。Gatewayレイヤで各サービスへ振り分ける。
	- https://azure.microsoft.com/en-us/blog/design-patterns-for-microservices/
- アーキテクチャ構成は`Go`, `Java`, `Python`。各サービスが`Docker`, `K8s`(`GCE`)上で動いている。サービス間は基本的に`gRPC`。
- API gatewayで認証やProtcol Transfomation（サービスごとに`json`に変換したり`gRPC`にしたり。）を行う。
- golangとgRPCの知識があればよい。フレームワークはつかってない。プロトコルバッファーの定義さえ書ければロジックに集中できる。RESTは構成などの設計が大変。
- Protcoll Buffer Definiton Management。全ての定義はひとつのリポジトリに記載されている。そのリポジトリを見れば各サービスのAPIがわかる。PRがマージされれば各サービスにクライアントコードが配布される。
- 動的なK8s環境上で`gRPC`を使ってロードバランシングするのは結構難しい。クライアント側でロードバランシングしないとk8のサーバがちゃんとラウンドロビンしてくれない。実例をあまり見ないで発表してほしい。
- ロギング。HTTPの場合は`middleware`だが、`gRPC`の場合は[`Interceptors`](https://github.com/grpc-ecosystem/go-grpc-middleware)を使ってる。
- [grpc-go](https://github.com/grpc/grpc-go)は`interceptor`をひとつしか使えないが、[`go-grpc-middleware`](https://github.com/grpc-ecosystem/go-grpc-middleware)を使えばチェーン関数を利用できる。
- 複数のサービスをまたがるプロファイリングはGCP stackdriver tracingから[`grpc-go client`](https://github.com/GoogleCloudPlatform/google-cloud-go/tree/master/trace)が提供されている。

---

各サービスのAPI定義をセントラルリポジトリに集約する運用はマイクロサービスの集合体としてサービス間の互換性を維持するため、各サービスの集合体としての全体仕様を可視化するために非常によい仕組みだと思った。`API gateway`レイヤで外部との連携や認証を一括処理してくれるのも、各サービスの開発者がロジックの開発に集中できたり、疎で有り続けるためのよい仕組みだと思った。



# 大規模で運用するためのモニタリングエージェントを自作する話

発表者：huydx

https://www.slideshare.net/dxhuy88/gocon-autumn-story-of-our-own-monitoring-agent-in-golang

- 内製モニタリングエージェントの話。各ホストの中で動いている小さなエージェント(だから`fluentd`などとは違う）
- ジェネリクスなエージェントではない。それぞれのメトリクスなどの構造を理解したエージェント。
- LL系はサーバごとにバージョンが違ったりするので、使うのが難しい。自分の会社だけのドメイン構造などがあるので、自作したい。
- 既存使用言語が「ぼくの◯◯、あなたの◯◯」という状態だったので、「みんなのGo」が書けるGo言語を利用してスクラッチすることに。
- 設計思想は`Input`, `Codic`, `Output`を自由に変更できるように。plugable
- シングルプロセス。プラグインモデル。コレクションモデルはプッシュではなくプル形式。
- 各エージェントの情報は[`Prometheus`](https://prometheus.io/) / [`Grafana`](https://grafana.com/)を利用してモニタリングしている。
- Golangはプロトタイピングがすごいラク。

---

LINEという多様なサービスの集合体でのモニタリングをするにはポータビリティ性の高さが必須、そこでGoが選ばれたとのこと。プラグインが大量の種類に作られそうなので、コードに個性が発生しにくいところも最適なのかもしれない。

# Prometheusから考察するGoのロギング

発表者:大和屋貴仁

https://www.slideshare.net/ssuser88ff5b/gocon2017go


- PrometheusはSRE本で紹介された最近Hotになったツール。異なるロギングライブラリを使用している。プラグイン製作者でも扱いやすそうな[`go-kit`](https://github.com/go-kit/kit)に一本化しよう。という話があった。
- 標準Logパッケージで不便な点。型で定義されているので、別の実装を提供しにくい。グローバルなオブジェクトを呼び出す関数が存在するので、制御しにくい。やっぱりレベリングがほしい。
- プロメテウスはgo-kit。key-value形式
- terraformは内製の`loguitls`。標準パッケージをラップしているちょっとモダンなやつ。
- K8sは[`glog`](https://github.com/google/glog)。googleが使っているC++と同じロギング実行。
- Grafanaは[`log15`](https://github.com/inconshreveable/log15)
- InfluxDBはuber-goの[`zap`](https://github.com/uber-go/zap)というlogging.

---

乱立しているGoのロギングの調査結果。awesome-goにも34種類のロギングツールが載っているらしい。レベリングについてはこんな感じのようだ。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">Dave 先生の <a href="https://t.co/APWJ2x3iux">https://t.co/APWJ2x3iux</a> には『誰も warn なんか見てねえだろ？ fatal も error も log 以前の問題が残ってる。だから level なんていらねー』見たいな論調で書かれている🤔 <a href="https://twitter.com/hashtag/GoCon?src=hash&amp;ref_src=twsrc%5Etfw">#GoCon</a></p>&mdash; 眼力 玉壱號 (@objectxplosive) <a href="https://twitter.com/objectxplosive/status/927043412888399872?ref_src=twsrc%5Etfw">2017年11月5日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



ハッシュタグの雰囲気的にgRPCでも使える`zap`が人気のようだった。Mercariも基本的に`zap`とのこと。
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">MercariのGoのAPIやToolは基本全てzapで統一されてる（少なくとも私の観測範囲では）基本はzapで良いかなと思う．Context awareにするのも自分でちょっと書けば良いし <a href="https://twitter.com/hashtag/GoCon?src=hash&amp;ref_src=twsrc%5Etfw">#GoCon</a></p>&mdash; Taichi Nakashima (@deeeet) <a href="https://twitter.com/deeeet/status/927040413722013696?ref_src=twsrc%5Etfw">2017年11月5日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>





# Goのcompileの仕組みと高速化への取り組み

発表者:niconegoto
https://speakerdeck.com/niconegoto/how-to-achieve-parallel-compilation-in-go-1-dot-9


- Go1.9から入った並列コンパイルで行われたことについて。
- だいたいこの辺の話。https://golang.org/src/cmd/compile/internal/
- 各コンパイル肯定への`mutex`の追加が変更のほとんど。
- [`AST`](https://ja.wikipedia.org/wiki/%E6%8A%BD%E8%B1%A1%E6%A7%8B%E6%96%87%E6%9C%A8)から[`SSA`](http://www.is.titech.ac.jp/~sassa/coins-www-ssa/japanese/ssa-nyumon.html)への変換時のパッケージ検索に使用されるところ、とか。
- コンパイラシンボルからリンカシンボルへのマッピング格納部分とか。
- `cmd/cimpile/internal`らへんのコードは結構雑な作りが残ってるのでパッチ送ろう。
- `cmd`以下は汚いので、読むのは覚悟がいる。

---

大学でコンパイルの講義を受けている友人がいたのでGoのコンパイラについて調べてみたとのこと。
コードを読むだけではなくて、その変更に関わるgerritなども一緒に読むのはかなり勉強になりそうだと思った。あえて泥臭そうな部分を選ばれた強い気持ちにも尊敬の眼差し。

# Async, Persistent, Fast, and Stable "Enough" Queue/Worker Using Go and PostgreSQL

発表者:_achiku

https://speakerdeck.com/achiku/gocon-2018-good-enough-async-job


- [バンドルカード](https://vandle.jp)というサービスのバックエンドは全てGoで書かれている。
- Queue/Workerに[`que-go`](https://github.com/bgentry/que-go)を使っている。
- `que-go`はrubyの[`Que`](https://github.com/chanks/que)のインポート
- データの変更とEnqueがトランザクションで守られている。
- EnqueはAPI Serverから`que_jobs`テーブルにインサートするだけ。worker-serverのDequeは普通のDeque。ジョブのロックは`postgeSQL`の仕組みでロックしている。
- `que_jobs`テーブルはジョブがどのジョブタイプか、どこでエンキューするのか、ジョブを実行するときのパラメータを保持している。
- Advisory-lockという`postgreSQL`の機能を使うと、アプリケーション側でlock時の制御を決められる。
- 1ジョブ1トランザクションが基本だけれども、外部APIも絡んでるとロールバックすればよいのかわからなくなるため、ジョブをカテゴライズした。
- 外部のデータを変更するようなジョブをどう対処するのかが問題だが、泥臭く実装している。


---


外部APIとはカード決済部分のはずなので、そのような融通がきかない外部APIとどう共存しているのかも聞いてみたかった。外部APIに引きづられるような性能要件はトライアンドエラーになる（動かしてみてはじめてわかることがある）と思うので、プロジェクトマネジメント的な部分も気になった。


# Diff algorithm in Go

発表者:cubicdaiya

- Diffを取るさいに必要な3つの技術。
	- 編集距離（Edit Distance） 追加と削除の総数を算出する。
	- LCS（Longest Common Subsequence）一番長い同一文字列
	- SES（Shortest Edit Script） 別の文字列に変換するための計算式。
- Gitとかは、Myers’s algorithm
- SVNは、Wu’s algorithm
- Golangで実装したWu’s algorithmの解説。


---

ちょっと眠くなっていた && 自分には難しくてメモが取れなかったので、詳細はご本人の解説へのリンクで。

https://qiita.com/cubicdaiya/items/2ce3dce3b771de5c3733



# Ebiten - Go のシンプルな 2D ゲームライブラリ

発表者:hajimehoshi

https://docs.google.com/presentation/d/e/2PACX-1vSSbSxPObBZcJHjvUpAt-HEJVLaux2FQBpJbvbxInJgmEhxSn-lVxTVxUMmUNQwtJtC8w6_HkhuW2hk/pub?start=false&loop=false&delayms=3000&slide=id.p


- 2DゲームのマルチプラットフォームライブラリをGoで実装している。https://github.com/hajimehoshi/ebiten
- 基本的なロジックは矩形から矩形への描画を行う。
- GopherJSはGoで書かれている。なので、自身で自身をJSに変換することができる。

- デスクトップはcgo使うことになるが[`GLFW`](http://www.glfw.org/)。
- ブラウザーは[`GopherJS`](https://github.com/gopherjs/gopherjs)。GoのなかでJSのコードが呼べて、GoのコードをJSに変換することができる。。`GopherJS`はGoで書かれている。なので、自身で自身をJSに変換することができる。
- モバイルは公式提供の[`gomobile`](https://godoc.org/golang.org/x/mobile)。
- `gomobile build`は実行ファイルを吐く。`gomobile bind`すると動的ライブラリが作れる。
- `build`は署名を入れるのがメンドくさい。`bind`は呼び出し元をネイティブ言語でつくればなんとかなる。広告などを出したかったら`bind`でやるしかなさそう。
- `gomobile`はバグの切り分けが難しい。また、`Context losts`というようなモバイル特有の問題も対応する必要がある。
- プロファイルツールが使えなかったりパフォーマンスチューニングが難しい。
- GPD pocket上のChromeやKindle Fire上のIssueが上がったりもしたが、ちゃんと動いている。

---

マルチプラットフォームが得意といえども、モバイルやブラウザを含めるとかなり大変とのこと。とは言え挙げられたIssueはちゃんと直しているらしい。バグの切り分けかたについても参考に聞いてみたかった。

# reviewdog and static analysis for Go

発表者:haya14busa
https://docs.google.com/presentation/d/1_BWQXamZvIhL3l9ziL9zb25yP9RjpgXoxkWX-48ECss/edit#slide=id.p

- reviewdog。https://github.com/haya14busa/reviewdog
- `linter`と`diff`の結果から、追加された部分のlint結果だけを出力する。CLIやPRコメントに結果を出力する。
- [`Hound CI`](https://houndci.com/), [`Side CI`](https://sideci.com/ja)といったSaaSはユーザーがlinterを選択できない。
- [`Pronto`](https://github.com/prontolabs/pronto)、`reviewdog`と似ているけど、Rubyで出来ているので、わざわざRubyの実行環境用意してrubyのプラグイン書かないといけないのもめんどくさい。
- このようなツールをつくると、`linter`のいろいろな出力フォーマットに対応しないといけない。
- `Vim`の[`quickfix`](http://vimdoc.sourceforge.net/htmldoc/quickfix.html)機能が似たような機能を実現している。[`errorformat`](http://vimdoc.sourceforge.net/htmldoc/quickfix.html#errorformat)は出力のパースにポインターやスタックを用意していて、リッチなlinterの出力にも対応できている。
- このエコシステムをGoに移植して、柔軟なエラーフォーマットに対応できるのがreviewdog。独自Linterをサポートすることもできる。

---

はやぶささんがgoにコミットしたときはコミットメッセージもレビューしてもらったらしい。
既存のエコシステムを自分のツールに取り込むのは自他にメリットがあって素晴らしい。


# LT

#パッケージ構成っていつでも悩ましい

発表者：keigodasu
https://www.slideshare.net/keigosuda/20171105-go-con2017lt-81642440


- Mercari Confの[Web アプリケーションにおける Go 言語のパッケージ構成 〜メルカリ カウル編〜](https://speakerdeck.com/mercari/ja-golang-package-composition-for-web-application-the-case-of-mercari-kauru)が読むのがよい。
- シンプル/未経験者も多いときはRailsライク、経験者が多いときはDDDライクに近づけていく。
- [`アンチパターン`](https://github.com/gophercon/2017-talks/blob/master/EdwardMuller-GoAntipatterns/GoAntipatterns.pdf)などを真に受けず、`utils`とか`api`という名前を使ってもよいのではないか（わかりやすいことが一番なのだから）
- あまりパッケージ分けの事例がないので、もっと公開されていよいかも

---

とはいえ`utils`はおすすめしないってつぶやきは多かった気がする。

# Consider better error handling for web apps

発表者：izumin5210

https://speakerdeck.com/izumin5210/consider-better-error-handling-for-web-apps

- 集約には[`Sentry`](https://sentry.io/welcome/)、[`HoneyBadger.io`](https://www.honeybadger.io/)などを使っている。
- レスポンス返す前に通知する場合、エラーが`Unexpected`かどう決めるか。レスポンスコードはどう決めるか。
- エラーが出たところで通知する場合、ロギングライブラリと密結合したり、どこでもかしこも通知が埋め込まれることになる。
- 基本的には`middleware`でエラー通知するのがベターなのかなと思っている。
- gRPCサーバのエラーハンドリングする例を作った。
	- https://github.com/izumin5210-sandbox/grpc-and-gateway-sample-app-go

---

サンプルアプリまで用意されている発表姿勢見習いたい。ボリュームも大きすぎないので`gRPC`よくわからない勢としては足がかりとして読ませていただく。

# Goだってパフォーマンスチューニングが必要です

発表者：usk

- GolangでもGCが間に合わないこともある。
- 事例。負荷テストかけたらメモリーリークで死んだ。
- ある1リクエストのメモリアロケートが他と比べ、20倍重かった。
- `slice`をサイズ指定をせずに`make`したり、`Structure`をやたら多用して`json.Marshal`を多用していたのが原因だったっぽい。
- Benchmarkと[`pprof`](https://golang.org/pkg/net/http/pprof/)で重い処理を確定した。

---

パフォーマンスチューニングはこの動画が最高らしい。
Profiling and Optimizing Go
https://www.youtube.com/watch?v=N3PWzBeLX2M


# バイオインフォマティクスにGo言語を持ち込んだ話

発表者：otiai10

https://speakerdeck.com/otiai10/bioinformatics-meets-go

- 良かった。簡単に書ける。十分早い。携帯性がよい。
- 悪かった。Bio infoだとGopherいないしライブラリもPythonとかばっかり。標準や定義が未整備分野だったので、構造定義が大変だった。
- Shape of life

---

ツール作成をするにおいてマルチプラットフォームなシングルバイナリを生成できることはかなり強みになる。
途中からかなりエモかった。

# Go でインタプリタを書いてみよう

発表者：knsh14

https://speakerdeck.com/knsh14/go-deintapuritawo-shu-itemiyou


- 人生で一度オレオレ言語作ってみたいところ、Goで良さそうな本出たのでやってみた。
- [Writing An Interpreter In Go](https://interpreterbook.com/)
- 遅い人でも１〜２ヶ月で出来る。
- ASTの気持ちになれた。
- エラー処理はGoのプラクティスに従わない。(そのほうがユーザーフレンドリーに処理できるらしい)

---

英語があまり読めなくてもなんとかなるとのコト。
一度オレオレ言語を作ってみると、Goがジェネリクスとか入れない理由（ASTが複雑になる）も身にしみてわかるのかな。。。？
