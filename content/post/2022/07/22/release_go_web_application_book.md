+++
title= "「詳解Go言語Webアプリケーション開発」という本を発売しました"
date= 2022-07-22T02:14:41+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go","book"]
tags = ["golang"]
keywords = ["詳解Go言語Webアプリケーション開発"]
twitterImage = "2022/07/mybook_cover.jpeg"
+++
「詳解Go言語Webアプリケーション開発」という書籍を執筆し、2022/07/22にC&R研究所様より発売しました。  
全国書店やAmazonで購入できます。本記事では本の内容の紹介や執筆経緯、執筆してみての感想など書きます。

https://www.c-r.com/book/detail/1462

<!--more-->


<iframe sandbox="allow-popups allow-scripts allow-modals allow-forms allow-same-origin" style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B0B62K55SL&linkId=6232a731c69b5376303b01029e455671"></iframe>

# 本の内容について
本著は大きく分けて二部構成になっています。  
第一部は次のようなテーマにまつわるトピックを中心にまとめました。

- 他の言語を経験していると不思議になるGoの言語仕様
- 最近Goを始めた方は知らない過去の経緯や歴史的背景

第二部ではGoを用いたWebアプリケーションのコードをハンズオン形式で解説しました。  
テストコードを書き段階的な変更を繰り返しながらAPIサーバの実装を試みています。

- Docker Composeを使ったローカル開発環境の構築
- [`github.com/cosmtrek/air`](https://github.com/cosmtrek/air)を用いたホットリロード開発環境の構築
- MySQLやRedisに依存したテストのGitHub Actions上での実行
- `go:embed`ディレクティブを使ってファイルの埋め込み
- ミドルウェアパターンと`context`パッケージで実装した`JWT`を用いた認証・認可の実装
- モック（[`github.com/matryer/moq`](https://github.com/matryer/moq)）を使ったテストの作成
- フィクスチャを用いたテストデータの作成


ハンズオンに関しては「まず素朴に書いたプロトタイプコード」をリファクタリングしながら機能追加をしていく進行にしました。  
「動くコードなのになぜダメなのか？」をフォローしながら進めていくので「どうしてこう書くのか理由を教えてくれるメンターがいない」というような方にはとくにオススメできるかなと思っています。

なお、DDDやクリーンアーキテクチャに類するパッケージ構成などの解説はあえてしていません。
これらについてはエンジニアそれぞれで主張がありますし、そもそも「汎用的なベストな解」はないと思っているため、本著では触れていません。ご了承ください。

ハンズオンで実装するAPIサーバについては次のリポジトリで公開しています。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/go_todo_app" data-iframely-url="//iframely.net/bYCjhBW?card=small"></a></div></div><script async src="//iframely.net/embed.js" charset="utf-8"></script>

# 執筆するにあたって
今回Twitterでお声がけいただいて執筆を始めました。  
元々、普段の業務で「こういうことが書籍に書いてあったら紹介するだけで説明が終わるんだけどな」「ブログ記事はあるけれど書籍で紹介されてないかな？」とたびたび思ってはいました。  
なので今回は「こういう本があったらいいな」と思う内容を執筆しています。


## エンジニアとしての執筆経験
また、私はSOFT SKILLSという書籍にエンジニアとしてのキャリア形成（セルフブランディング）をかなり影響されています。

<iframe sandbox="allow-popups allow-scripts allow-modals allow-forms allow-same-origin" style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B09S3J33NR&linkId=c941ee441a189af4e4047660a4082186"></iframe>

ブログ活動や登壇活動もこの書籍を読んでから頑張るようになりました[^softskill]。  
実は執筆のお話をいただく以前にUdemy講師のお話を受けたりしたこともあったのですが、そのお話は諸々の事情で断念していました。  
なので今回頂いた執筆の機会は何がなんでも出版すると決めていました。

[^softskill]: 技術書の執筆はおカネ目的ではできないこともこの本で知っていました。

# 執筆してみた感想
今まで技術書典で合同誌に寄稿という形で何回か原稿は書いていたり、ブログを1年以上毎週書いていました。  
なので文章のアウトプットには多少自信があったものの、はじめての商業誌、はじめての単著での執筆、育児と両立というところで今までの経験とはまったく異なる体験になりました。

## ページ数が増えない
ページ数を目標においては本来いけないのでしょうが、書籍として妥当なボリュームで販売するために目標ページ数は300ページ以上のような形に決まっていました。  
「コードを書けば自然にページ数は増えていく」「ブログ記事を書くようにストックした複数トピックを合算すればページ数が増えていく」みたいな浅はかな考えは間違えでした。  
思った以上に1文で悩む時間も多く、やっと1つトピックを書き終わっても予想のページ数の半分以下になることも多かったです。  
技術書典の合同誌のときは多くて10ページ程度の執筆だったので、300ページは途方もなく遠いゴールでした。

## 紙媒体という登壇やブログ記事と違った難しさ
今回改めて痛感したのは登壇やブログ記事と異なる紙媒体でアウトプットを出すという緊張感でした。  
登壇はメインが文字ではなく口からの発表なのである程度用語の使い方が不正確でもニュアンスが伝わったりします。  
また、聴講者の様子やフィードバックから伝わっていなことを察したときは補足説明ができます。  
ブログはリライトが何度も可能なので、ひとまず公開してあとで校正するという選択肢が取れます。  
紙媒体での出版は上記の試行錯誤をとるアプローチができないので一文一文の緊張感がかなり違いました。

## 執筆時間の確保
副業にも通じる話かもしれませんが、執筆時間を確保するのが今回の執筆の中の一番の課題でした。  
発売を半年延期[^pub_date]させていただいており、各所の調整をしていただいたC&R研究所の皆様と予約していただいた皆様には申し訳なさとお待ちいただいた感謝しかありません。  
言い訳がましいのでブログでは詳細を省きますが、双子を育児しながらの執筆はかなりハードルが高かったです。
保育園に入園すれば時間が作れると思っていたのも間違えでした。

[^pub_date]: 当初は2022年2月発売予定でした。

# 今後について
締切の都合上書籍に含められなかったトピックも多くあります。  
それらに関しては補遺と立ち読み見本ページを兼ねてzenn bookで近日中に公開したいなと思っています[^2]。  
書籍購入後知りたいコトが載っておらず不満を持たれた方、購入を迷われている方は少しお待ちください。

また、私生活の余剰時間をすべて執筆時間に当てていてブログを書く時間がなかったなので今月からはこのブログ活動も再開したいです。

[^2]: 本当はこの記事と同時公開の予定でしたがいろいろな事情でここ数週間は執筆時間が（本業の勤務時間すら）確保できませんでした。

# 終わりに
今回出版するに当たり多くの方々にレビューのご協力をしていただきました。  
レビュー時期までに執筆が間に合わず当初の依頼形式と異なるレビュー実施になってしまったのですが、それでも多くのご助言・ご指摘をしていただいたレビューアの皆様には感謝してもしきれません（そして大変勉強になりました）。

最後に、本書が少しでもみなさんのGoを用いた開発とGoコミュニティの発展へお役に立てれば幸いです。  
なお、書籍の内容に対する指摘、質問をしていただける方はGitHub DiscussionsかTwitter（[@budougumi0617](https://twitter.com/budougumi0617)）からしていただけると幸いです。
https://github.com/budougumi0617/go_todo_app/discussions



<iframe sandbox="allow-popups allow-scripts allow-modals allow-forms allow-same-origin" style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B0B62K55SL&linkId=6232a731c69b5376303b01029e455671"></iframe>

また、PDF版、EPUB版をご希望の方はC&R研究所様ECサイトの`本の森.jp`でご購入ください。
https://book.mynavi.jp/manatee/c-r/books/detail/id=131170
<div class="iframely-embed"><div class="iframely-responsive" style="height: 170px; padding-bottom: 0;"><a href="https://book.mynavi.jp/manatee/" data-iframely-url="//iframely.net/jKDgHkZ"></a></div></div><script async src="//iframely.net/embed.js" charset="utf-8"></script>