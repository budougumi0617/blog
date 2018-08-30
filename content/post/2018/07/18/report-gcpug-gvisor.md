+++
title = "GCPUG Tokyo gVisor Day July 2018 参加メモ #gcpug #gvisor"
date = 2018-07-18T08:30:00+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["report", "gcp"]
tags = ["gcpug","gcp", "gvisor"]
+++

[GCPUG(Google Cloud Platform User Group)](https://gcpug.jp/)の勉強会に参加してきた。  
先日OSSとして公開されて話題となっているgVisorの話を聞いてきたので参加メモ。

<!--more-->

- https://github.com/google/gvisor
- コンテナの軽量さと、より安全な分離を実現する「gVisor」、Googleがオープンソースで公開
  - https://www.publickey1.jp/blog/18/gvisorgoogle.html

|||
|---|---|
|URL|https://gcpug-tokyo.connpass.com/event/90909/|
|会場|六本木ヒルズ 森タワー|
|日時|2018/07/12(木) 19:00 〜 21:30|
|ハッシュタグ|[#gcpug](https://twitter.com/hashtag/gcpug)|


# 所感
本題のgVisorについてはもちろん、Linuxのセキュリティ機能の知識、企業が公開しているOSSからその企業の戦略を推測する新しい見方を知ることができた。

最初の[@yuryu](https://twitter.com/yuryu)さんの発表では以下を聞くことができた。

- 信頼性が低いアプリ・プロセスを実行するときに守るべきLinuxのセキュリティ
- gVisorがどのようにしてセキュアなコンテナランタイムを実現しているか

まさかPaaSの勉強会でLinuxのセキュリティ機能について教えてもらえると思っていなかった。  
どんな点に気をつけなければいけないのか、を「非特権ユーザでサービス実行する」というところから丁寧に教えてもらえたのでその後の話の重要性もかなり噛み砕いて納得することができた。自分は雰囲気でLinuxを動かしている(本番はSREの方々がいい感じのAnsibleを流したり、AWSの設定をしてくれているし、ローカルで試すときはまずSELinux切る)ので、ちゃんと知っておかないといけないなと思った。  
gVisor自体についても話を聞くのは初めてだった。正直「googleのすごい人たちがGoでLinux作り直しているのかな？」くらいの認識だったので、どんな思想で作られたOSSなのか知ることができて良かった。

- 不特定多数の任意のアプリを安全に動かせるコンテナランタイムの実現
  - google(GCP)はPaaSとして開発者すら意図してない挙動をする可能性があるアプリをホスティングしないといけない
- Linuxの再実装をしたいわけではない(gVisorでシステムコール全てを再現するつもりはない)

また、デモでは1月に話題になったメルトダウンを使ったコンテナからホストへのハックを実際に見せてもらい、gVisorがそれを防げることを確認できた。

[@apstndb](https://twitter.com/apstndb)さんの発表ではgVisorがどのようにGCP内で使われているのか知ることができた。  
gVisorの動作確認済み言語、アプリからGCPの動きを推測するのは「その発想はなかった！」という感じだった。  
たしかにGCPや何らかのサービスに利用するために開発しているはずなので、gVisorの設計やサポートしている機能からgoogleの戦略を考えるのは非常に面白い話だった。


以下自分用メモ。

# gVisor 入門: サンドボックス化されたLinuxコンテナランタイム
[@yuryu](https://twitter.com/yuryu)

なぜgVisorが必要だったのか。Dockerなどの既存のコンテナ技術の流れから何が違うのか。

## コンテナについて
- 13年にDockerが公開してからコンテナ技術が世界に広まった
- googleは自社サービスで40億以上のコンテナを毎週起動している
- 既存のコンテナ技術では完全にホストOSから隔離されてない
  - "Containers do not contain"
    - https://www.slideshare.net/d0cent/containers-do-not-contain

## Linux OSとセキュリティ
- 大昔の各サービスプロセスが`root`(特権ユーザー)で動いていた時代
  - なんでもできちゃった。無関係なプロセスを終了させたりもできた
  - 悪意のある行動がしやすい状態になっている
- 非特権ユーザーでサービスプロセスを実行するようになった
  - いろいろ制約をつけられるようになった
  - まだ`Raw Socket`や`setuid`という抜け穴がまだある
    - https://ja.wikipedia.org/wiki/Raw_socket
    - https://ja.wikipedia.org/wiki/Setuid
- `capabilities`を使って更に制約を課すことができるようになった
  - http://man7.org/linux/man-pages/man7/capabilities.7.html
  - 細分化された権限（`CAP_SYS_NICE`、`CAP_SYS_CHROOT` etc...)sを使って必要最低限の権限のみを渡すことができるようになった。
- ここまででもまだ各サービスプロセスはCPUやメモリなどの計算資源を際限なく利用できる
  - `cgroups`でリソースの制約をつける
    - https://ja.wikipedia.org/wiki/Cgroups
- これでも他のプロセスなどを見れる。
  - `namespace`で名前の分離のための名前空間を作る
    - http://man7.org/linux/man-pages/man7/namespaces.7.html
  - これで各サービスプロセスの`/`ディレクトリを分離できる
- これで十分な分離ができたのか？
  - まだだめ。ホストカーネルやデバイスドライバを共有している
    - 単一の脆弱性が特権昇格や情報漏えいが発生してしまう可能性
    - ネットワークスタックにも脆弱性がある。


## 安全なコンテナを作る
- 上記のセキュリティの問題をクリアした安全なコンテナを使いたい
  - コンテナ内からホストに影響が及ぼせないこと
  - 通常のコンテナと同じくらい簡単に使えること
    - アプリ側にコンテナの変更に伴う修正が不要なこと

## gVisorの設計思想
https://github.com/google/gvisor#how-is-gvisor-different-from-other-container-isolation-mechanisms

- 参考になる既存世界1: 仮想マシン
  - ハードウェアをエミュレートして完全なOSを実行
  - 優れた分離性、互換性
  - 高いオーバーヘッド、メモリ使用量
  - 固定されたCPU/メモリ割り当て
- 参考になる既存世界2: ルールに基づいたアクセス制御
  - `seccomp`, `SELinux`, `AppArmor`
    - http://man7.org/linux/man-pages/man2/seccomp.2.html
    - https://ja.wikipedia.org/wiki/Security-Enhanced_Linux
    - https://ja.wikipedia.org/wiki/AppArmor
  - ネイティブに匹敵する非常に良い性能
  - ルールの定義、設定が大変
  - 未知のバイナリに対応しづらい
- ２つの世界の良い点をいいところどりする
  - 独立したカーネル
  - ソフトウェアの仮想化
  - 仮想化されたハードウェアインタフェースは柔軟性がない
  - OSをそのまま再現するのはでかすぎる
  - コードが2,0000万行を超えるLinuxカーネルは（抜け穴が多く）安全性が低い
  - サンドボックス化は攻撃面を減らす効果的な手法

### gVisorの概要
https://github.com/google/gvisor

- gVisor上のコンテナランタイムはユーザーモードで動作する小さなカーネル
  - gVisorのカーネル内でアプリからのシステムコールをトラップし実行する
    - gVisorがホストのシステムコールを実行する。アプリはgVisorしか操作してない
- 通常のプロセスのような柔軟なスレッド、メモリ割当
- 仮想化と比べて低いオーバーヘッド
- Linuxシステムコールをユーザー空間内で独立して実装している
  - 現在211のシステムコールを実装している
  - アプリ側はとくに意識することがない
- 最初からセキュア
- gVisor上のコンテナひとつひとつが別々のユーザーモードカーネル上で実行される
- メモリ安全、型安全なGoで書かれている。(Goで実装された理由はGoLoverなのも大いに関係している）
  - https://github.com/google/gvisor#why-go

### gVisorのアーキテクチャ
https://github.com/google/gvisor#architecture

- gVisorのコンテナランタイムは2つの別々のプロセスで動く
- Sentry システムコールをエミュレートしている
- Gofer ファイルアクセス
- ２つのプロセスは9Pプロトコルで通信している
  - https://ja.wikipedia.org/wiki/9P
- ネットワークはSentry内のユーザーモードで動いている

- なぜ２つに分けているか
  - もっとも悪用されるのは`socket`と`open`
  - ファイルシステムをGofer経由でしかアクセスできないようにすることでSentryに何かされても安心
- システムコールのトラップ
  - ptrace PTRAP_SYSEMUを使ってシステムコールをトラップしている
  - これは全Linuxで動くという点が利点。
  - KVM（試験的）
  - gVisorがVMMM件ゲストOSとして動作
  - ハードウェアによる仮想化サポートが要件

## gVisorの性能と用途
- メモリ使用量15MB
- 起動時間150ms
- システムコールに若干のオーバーヘッド(コンテナよりおそい)
- gVisorが向いていない用途
  - ホストと異なる種類のゲストOSを動作させたいとき
  - 完全に信頼されたバイナリ（だけを実行するなら普通のコンテナでよい）
  - システムコールを多用するアプリ
  - 完全なアプリケーションの互換性を期待する
- Linuxの再実装をしたいわけではない
  - gVisorでシステムコール全てを再現するつもりはない
- gVisorで動くアプリ
  - https://github.com/google/gvisor#what-works
  - golang, java, php, node, python etc...
- gVisorはクラウドの裏方
  - GAEはgVisorで動作している
- OSカーネル自体の研究開発に向いているかも？
  - gVisorにGoで変更を加えユーザーモードで実行する
  - gVisorで試してみてLinuxへ移植するとか
  - Cで実装された巨大なLinuxのコードよりはユーザーフレンドリーなはず？
- gVisorは安全なサンドボックス内でコンテナを実行する新しい手法
  - 裏方の技術を知ることは正しい判断する上で大切

## gVisorを実行する方法
- https://github.com/google/gvisor#installation
  - Dockerで動かしたいときは`docker run`に`--runtime=runsc`をつける
  - 最後にメルトダウンを悪用するDockerコンテナが、gVisor上ではCPU情報へアクセスできなくなるデモをされていた
    - https://ja.wikipedia.org/wiki/Meltdown
  - k8s上で動かしたいときはcrio.confに一行たすだけ。


## まとめ
- gVisorを使ってサンドボックス内で安全にコンテナを実行できる
- コンテナのメリットは保たれている
- コンテナとホストOSの間に厳格な境界をもたらす
- 信頼されていないバイナリをコンテナ内でより安全に実行できる
- 裏方の技術を知ることは正しい判断する上で大切
- 今から5分くらいで試せるよ！
  - https://github.com/google/gvisor


#	gVisor と GCP
[@apstndb](https://twitter.com/apstndb)

- gVisorとGCP
  - https://docs.google.com/presentation/d/1F6k6bBS7BOUQWl9WGEpQJDyfvd04Et_-EeIHRoQzz-Y/edit

- gVisorに対する観点の違い
  - googler勢、システムプログラミング勢、マルチテナント勢で視点が異なる
  - GCP勢はもっともgVisorからメリットを享受していそう。

## App EngineとgVisor
- GAEのような任意のプログラムを実行できる環境では悪意あるユーザーの攻撃が多方面に及ぶ
  - 任意のシステムコールが呼べる状態では危険
  - 信頼出来ないプログラムはサンドボックス上で動作させたい
- 従来のGAEはランタイムごとに実装されていた。例えば魔改造されたJVMとか
- gVisorで処理系に改造より安全なサンドボックスになる
- GAE/SEのランタイムがどんどんgVisorによって増大している。Java8やNode.js8、Python、PHP7...

## App Engine以外とgVisor
- Cloud functionもgVisor
- `faster than light`(FTL)の対応でどんなランタイムが増えていくのかわかりそう
  - https://github.com/GoogleCloudPlatform/runtimes-common/tree/master/ftl
  - gVisor上で走るOCIイメージのビルドツール
  - GAE/Node.jsのビルドログで確認可能
- FTLの対応言語がGAEと同じ。pythonとかPHPとか
  - https://github.com/google/gvisor#what-works
- GCPがサポートしている７つの言語が最優先なのでは？
  - https://cloud.google.com/docs/?hl=en
  - となると次は.NETやRubyが…？
- gVisorはサーバーレス戦略と密接な関係がありそう

## gVisorからみるgoogle, GCPの今後
- gVisorの対抗となるVMベースコンテナランタイムであるKata Containers側からは批判も出ている
  - https://katacontainers.io/
  - 「既存技術を危険危険いうのはFUDしているのでは？」
- Googleの特殊なトレードオフを把握する
  - 互換性
  - リソースフットプリント
  - セキュリティ境界の頑強性
  - パフォーマンス
- 自社でコントロールできるOSSじゃないといざというとき責任が取れないというのもありそう
  - コミュニティで機能を取捨選択するようだと難しい？
- googleにとってはリソース効率もかなり重要視されている
  - googleは100%再生可能エネルギーで電力を調達している
  - https://jp.techcrunch.com/2018/04/05/2018-04-04-google-matches-100-percent-of-its-power-consumption-with-renewables/
- gooogleによるgVisor動作確認済みテストについて
  - https://github.com/google/gvisor#what-works
  - 動作確認済みのものはGoogle社内かGCPでの優先度が高いのでは？gVisorのシステムコールの実装優先順にも関わるはず
  - nginxは最近まで動かなかった
  - GCEで提供されているDB系統(redis, mongo, mysql)が動作確認済みなのは意味深


# 関連
- [GCPUG Tokyo Container Builder Day February 2018 #gcpug 参加メモ](/2018/02/05/gcpug-container-builder-day/)
- [[k8s]GCPUG Tokyo Kubernetes Engine Day April 2018参加メモ #gcpug](/2018/04/28/gcpug-tokyo-kubernetes-engine-day-april-2018/)
