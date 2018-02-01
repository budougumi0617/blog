+++
title= "[Go]golang.tokyo #12 参加メモ #golangtokyo"
aliases = ["/2018/01/29/golangtokyo-12/"]
date= 2018-01-31T09:30:08+09:00
draft = false
slug = ""
categories = ["report","go"]
tags = ["golang","golangtokyo","k8s"]
author = "budougumi0617"
+++


golang.tokyo #12にブログ枠で参加した。  
今回はテーマ：『スタートアップ・新規事業におけるGo』で本発表2名、LT3名という内容だった。


|||
|---|---|
|URL|https://golangtokyo.connpass.com/event/76540/|
|会場|株式会社ネクストカレンシー（DMMグループセミナールーム）|
|日時|2018/01/29(月) 19:10 〜 22:00|
|Toggeter| https://togetter.com/li/1194466 |

# Next CurrencyとGAE/Go 
https://speakerdeck.com/sonatard/next-currency-gaego  
[sonatard](https://twitter.com/sonatard)

スタートアップであるNext Currency(NC)社がどのように技術選定してきたか。スタートアップとGAEの相性の良さについて

- プロダクトの開発をスタートアップのような人数で行うにおいて、如何にDB設計、API設計、ビジネスロジックのコーディングに注力するのか。
- 基本方針はSimple（理解が容易）になること。Easy（作業量を減らす）を目的にしない
- GAEは１つのアプリケーションが１プロジェクト
- さらにサービスという単位でアプリを分割し、サービスごとに異なるインスタンス構成にできる。（サービス別にデプロイすることになるので分けすぎ注意）
- NCではマイクロサービス化まではしていない。まだ個別にオーナーシップ(個別の技術選定等)をとる開発規模ではないから（エンジニア３名）
- アーキテクチャに正解はないが、決めることに価値がある。あるアーキテクチャを採用したときは、その採用目的を達成しなければならない。
  - 例：テスタビリティを上げるためにレイヤー分割、DIを導入したらならば、必ずテストを書く
- NCではドメインロジックを重点的にテストするためにアーキテクチャのレイヤーを設定した。
  - ドメインロジックのテストでHTTPリクエストを作りたくない
  - DB準備したくない
  - 大量のモック準備したくない
- 利用しているGoのパッケージは前述の通り基本シンプルなもの。
  - [go-chi/chi](https://github.com/go-chi/chi)
  - [go-gorp/gorp](https://github.com/go-gorp/gorp)
- APIについてはProtocol Bufferから独自にREST APIを自動生成している。アプローチが似ている[twitchtv/twirp](https://github.com/twitchtv/twirp)も気になる。
  - gRPCが使いたい理由がソースコードの自動生成だった（gRPCでなくてもよかった）
  - StackDriverのログが見やすいという副産物も生まれた
  - twirpはまだGoのコードしか作れないので、クライアント側があるNCではちょっとまだ難しそう
- APIはリリース直前に一番仕様が網羅される。最初から決めるのは無理なので大きな変更も結構している
- [Postman](https://www.getpostman.com/)を使ってAPIの叩き方をチームで共有している
  - 環境変数が使えるので、気軽にアクセス先のURL（ステージング環境）を変更できる
  - 変数もつかえるのでTokenをそのまま次のRESTに入れたりもできる
  - 問題切り分けも簡単になるし、Protocol Buffersの定義ファイルと併用すればSwaggerがいらないレベル
- テストは[docker-appengine-go](https://github.com/mercari/docker-appengine-go)を独自改造してCircleCIで行っている
  - Circle CI(config.yaml)、Docker Image、shell script、Makefileに適切に責務分割
  - Circle CI上でのツールのインストールはテストが遅くなるので避ける
- テストで使っているパッケージは[google/go-cmp](https://github.com/google/go-cmp)くらい
  - Table Driven + SubTest
  - モックが極力必要にならないような設計を心がける
- ログはBigQueryで。datastoreからのインポートは[favclip/ds2bq](https://github.com/favclip/ds2bq)

ここには書いていない細かいTipsなども多くあって、非情に参考になった。だいたいスライドに記載されているので、あとで読み返せばわかるようになっているのも非情にありがたい。
Postmanは便利なcurlのUIツール、としか考えていなかったけどチームで共有すると確かに便利だ。
レビューで動作確認するときも使えそう。テストやアーキテクチャ設計について、とても共感できることが多くて、自分も同じ意識を持って開発したいと感じた。


# GoとGCPとkubernetesを使ったMF KESSAIの歴史
https://speakerdeck.com/shinofara/gotogcptokuberneteswoshi-tuta-mf-kessaifalseli-shi  
[shinofara](https://twitter.com/shinofara)

2017年3月設立のマネーフォワード子会社、MF KESSAIのサービスをGo/GCP/kubernetes(k8s)でローンチ、運用するまでの話。

- MF KESSAI。2017年3月設立のマネーフォワード子会社。
  - https://mfkessai.co.jp/
  - データの可視化により資金繰り改善にフォーカスした会社。
  - MF KESSAIは請求業務プロセスを一括代行するサービスを提供。与信調査から代金回収まで。
- sinofaraさん自身は2016年にオンプレ中心だったのMFのクラウド・コンテナ化をするとして入社し、17年3月にMF KESSAIの責任者に。

## 創業初期、
MFのオペレータが利用する社内ツールとして開発をスタート。最初はエンジニア2名。

- Railsエンジニアが多いMFグループの中で型が厳格で低学習コスト使いたいと思い、Goを採用。コードが同じようになりやすいのも利点
- Goのパッケージは必要に応じて拡大していく方針で順次増やした。test/lint/vetは最初から導入。
  - [go-chi/chi](https://github.com/go-chi/chi)
  - [gorilla/csrf](https://github.com/gorilla/csrf)
  - [uber-go/zap](https://github.com/uber-go/zap)
  - [unrolled/render](https://github.com/unrolled/render)
  - [jinzhu/gorm](https://github.com/jinzhu/gorm)
  - [gorilla/schema](https://github.com/gorilla/schema)
  - [gorilla/context](https://github.com/gorilla/context)
  - [gorilla/sessions](https://github.com/gorilla/sessions)
- Dockerにより、カプセル化や冪等性を得た。
- GCPはGSuite導入済みだと権限管理がラク。無意識にデータが蓄積できるStackDriverと当時唯一k8sをサポートしていた。
  - https://cloud.google.com/stackdriver/?hl=ja

# 導入企業向けサービス開発期
Gopher 2名、フロント1名増員
導入企業向けWebサービス、導入企業向けREST API、内部APIとしてgRPC APIの開発、３つの開発を並列で走らせた。

- gRPC APIの開発。社内ツールとして作ったロジックを利用するインターフェース部分を主に開発。
- 社外向けのREST APIにはgoa、ゲートウェイには[Kong](https://getkong.org/)を採用
- 追加したGoのパッケージ
  - [grpc/grpc-go](https://github.com/grpc/grpc-go)
  - [golang/protobuf](https://github.com/golang/protobuf)
  - [godesign/goa](https://github.com/godesign/goa)
  - [jinzhu/configor](https://github.com/jinzhu/configor)

# 現在
Gopherが更に１名入社して合計6名。インフラ１名・インフラ１名・アプリ４名。
基本機能は実現できたので、サービスとしての価値の向上を行っている。

- Goは1.9。glideだったがdepに移行
- 今までは直列でPDF生成をPub/Sub + GAEでスケーラブルな状態に。FunctionsはGo未サポートのため未採用
- 既製技術は積極的に使う方針のため、Gateway/OAuth Endpointは[Kong](https://getkong.org/)。
- 各UI、APIとgRPC間のサービスメッシュには[Linkerd](https://linkerd.io/)を使っているが、最近は[Istio](https://github.com/istio/istio)も考えている
- GCP全体のモニタリングは[StackDriver](https://cloud.google.com/stackdriver/?hl=ja)。GKEの中のモニタリングは[DataDog](https://www.datadoghq.com/)。エラー通知は[Sentry](https://sentry.io/welcome/)で管理。もろもろの管理は時前でするのは辛いので、外部サービスに。

# Goを新規開発で使ってみて
- コンパイルすることで大胆な変更の中で起きる破壊的な状態を検知できる
- 標準でサポートされる各種静的解析ツールによるレビューコストの軽減
- 一定は同じ実装パターンになる、疎結合になりやすい言語設計によりチーム開発に向いている。
- 「前使っていた◯○言語（あるいはフレームワーク）で出来たXXXXみたいなことしたい！」という意見はやはり出るので衝突はある
- やはりリッチなUIは大変。。。
- それなりにコーディング量は増えがち。とくにif err != nil {...を量産するのでIDEとか馴染んでないと大変。

親会社の社内ツールとしてスタートし、ビジネスロジックの検証をしてからスケールさせていくというアプローチはとても堅実的でよいと思った。  
予想以上にメンバーが少ないところにも驚いた。また、自分は今年まさにこのGo/GCP/k8s/gRPCの構成の勉強を始めたので、構成を少し真似させてもらって軽く動くも作ってみたい。  
if err地獄について、自分はvimでgoを書くが、vim-goとsonictemplate.vimのスニペット使っているので、そんなに気にならない。VSCodeとかも`iferr`スニペットがあったはず。

- [mattn/sonictemplate-vim](https://github.com/mattn/sonictemplate-vim)
- [fatih/vim-go](https://github.com/fatih/vim-go)

## kushiの紹介
https://speakerdeck.com/cnosuke/golang-dot-tokyo-number-12-lt-kushi  
[#golangtokyo のLTでkushiというのを作った話をしてきた](http://cnosuke.hatenablog.com/entry/2018/01/31/114657)  
[cnosuke](https://twitter.com/cnosuke)


- クラウドを利用して開発していると、SSH Port fowarding使うことが多いのではないか
- そうなると、`~/.ssh/config`がすごいでかくなってくる
- kushiっていうツールを作った。みんなで`~/.ssh/config`を共有するツール
  - https://github.com/cnosuke/kushi
- S3とかに置いたクラウドの設定ファイルを取得して、手元でSSHトンネルをはってくれる

自分の業務用PCの~/.ssh/configも確かにごりごり書いてあった。  
また、いつの間にか「会社オフィシャルconfig」が更新されていてつながらなくなったりするので、そういう鮮度の問題も解決できそう。  
「金融系はクラウドベンダーを理由に落ちたりとか言い訳できない。最悪の場合あ別のベンダーでサービスを継続させるためにGCPとAWSをつかっている」という余談は流石きびしい…！という一言だった。
  
[THEO](https://theo.blue/)というサービスのSREをされているらしいのだが、開発環境がGCP。本番がAWSとのこと。
金融系はクラウドベンダーを理由に落ちたりとか言い訳できないので、別のベンダーでサービスを継続させるためにGCPとAWSをつかっているらしい。なるほど。。。！

## AWS Lambda Go First Impression
https://speakerdeck.com/timakin/aws-lambda-go-first-impression  
[timakin](https://twitter.com/__timakin__)

- AWS公式サポートのaws-lambda-go
  - https://github.com/aws/aws-lambda-go
- 今までGoをAWS Lambdaで使うときはApexを経由して実行していた。
  - http://apex.run/
- 構造的にほとんど違いがないので、Apexからaws-lambda-goへの移行はラク
- AWS曰く、積極的にグローバル変数で初期化をしようと言っている。
  - [Using Global State to Maximize Performance](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/go-programming-model-handler-types.html#go-programming-model-handler-execution-environment-reuse)
- ContextのなかのContext（GAEなども似たようなアプローチをしている）  
https://github.com/aws/aws-lambda-go/blob/28a6e9fa1fc02d45d9a8c0a8d22c7d5c3580aa03/lambdacontext/context.go#L80-L89
<script src="http://gist-it.appspot.com/https://github.com/aws/aws-lambda-go/blob/28a6e9fa1fc02d45d9a8c0a8d22c7d5c3580aa03/lambdacontext/context.go?slice=79:89"></script>
- vaidateArgmentsという情熱的なリフレクション  
https://github.com/aws/aws-lambda-go/blob/6e2e37798efbb1dfd8e9c6681702e683a6046517/lambda/handler.go#L37-L51
<script src="http://gist-it.appspot.com/https://github.com/aws/aws-lambda-go/blob/6e2e37798efbb1dfd8e9c6681702e683a6046517/lambda/handler.go?slice=36:51"></script>
- aws-lambda-goの実装コードは忍術みたいな実装をしていて、読むと勉強になる

今月のホットトピックのひとつ、AWS Lambda GoについてのLTだった。  
当日準備された内容ということだったが、充実過ぎる内容！自分も普段からちょっと話せる技術ネタのストックがあるようでないといけないなーと思った。  
手前味噌だが、自分が書いたaws-lambda-goの記事は以下。  
[AWS Lambda Go早めぐり(LambdaContext, Logging, Error...)](/2018/01/17/hello-aws-lambda-go/)

## SSM+yamlを使って開発別に 暗号化したDBパスワードを読み込む
https://speakerdeck.com/sioncojp/yamlssm-sample  
[@sion_cojp](https://twitter.com/sion_cojp)

- DBの情報（Railsでいうdatabase.yml的な情報）どうやって管理している？
- いくつか方法はあるが、サーバならまだしもコンテナだといろいろつらい。
- AWS Systems Manager(SSM) パラメータストアを使おう！
  - https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/systems-manager-paramstore.html
- 安価で文字列を暗号化してパラメータとして保持できる
- yamlにSSMパラメータを埋め込んだらdecryptさせるライブラリを同僚の方が作った
  - https://github.com/suzuken/yamlssm
  - yamlの中にssm://xxxxという文字列があればSSMで置き換える。
- docker-composeで作ったサンプル
  - https://github.com/suzuken/yamlssm

ご本人のブログにも詳しい使い方が書かれているみたい。  
[Go - AWS SSM Parameter Storeのデータを復号化とmockテストの書き方](http://sioncojp.hateblo.jp/entry/2018/01/15/212921)

timakin-sanも似たようなツールを作られていたとつぶやいていた。曰く「暴力的にSSMの値をenvに書き込むpackage」…！  
https://github.com/timakin/ssm2env

自社でも「コンテナ内で認証情報を使いたいけどdockerイメージ内に認証情報を置きたくない」という話があったので、こちらも身近な話題に感じた。  
~/.ssh/configについてもそうだが、こういう多数が結構困るあるあるネタを解決するツールを作るっていうのはよいなーと思った。自分は「何か作ってみたいけどネタがない」と言う感じの状態なので、こういう慣れてしまった身近な不便をもっと気にしていかなければ。

# その他
そろそろGo 1.10リリースされるので、golang.tokyoもリリースパーティの開催を予定中。

# 参加して
スタートアップとGoということで、小さく、少人数で行うGoの開発について聞くことができた。  
これは個人開発をするときにも非常に参考になるなと言う内容だった。本発表のお二人のお話を聞くと、シンプルに作ることを目指していてそれがGoを使う理由、Goが使われる理由にもなるんだろうなという気持ち。  
そしてそれを後押しするGCPという巨人。途中でも書いたが自分の今年の目標はGCP/k8s/gRPCなのでのんびりせずに早くGCPにデプロイしないとという気持ちになった。次回（通常回？リリースパーティ？）も参加するぞ。

### 関連記事
- [[Go]golang.tokyo #11 参加メモ #golangtokyo](/2017/12/12/golangtokyo-11/)
- [AWS Lambda Go早めぐり(LambdaContext, Logging, Error...)](/2018/01/17/hello-aws-lambda-go/)

