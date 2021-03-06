+++
title= "[書評] Engineers in VOYAGEを読んで事業に立ち向かう技術者を知る #voyagebook"
date= 2020-08-16T09:54:34+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["review"]
tags = ["book","voyage"]
keywords = ["Engineers in VOYAGE","書評","レビュー","エンジニアリング","voyage"]
twitterImage = "2020/08/16_voyagebook.jpeg"
+++

随所で話題のVAYOGE本を一気読みしたので感想をメモしておく。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">楽しみにしていた本が届いたので早速読む！！ <a href="https://twitter.com/hashtag/voyagebook?src=hash&amp;ref_src=twsrc%5Etfw">#voyagebook</a><a href="https://t.co/ogxcPIz1ED">https://t.co/ogxcPIz1ED</a> <a href="https://t.co/YuJFcfCx3z">pic.twitter.com/YuJFcfCx3z</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1293813474879889408?ref_src=twsrc%5Etfw">August 13, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<!--more-->

# どんな本なのか
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.lambdanote.com/products/engineers-in-voyage" data-iframely-url="//cdn.iframe.ly/TCGeRhB"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

本はVOYAGE GROUPの各事業会社のエンジニアの話がまとめられた本だ。  
6つの章はすべて [@t_wada][t_wada] さんとVOYAGE GORUPエンジニアのインタビューとして、歴史の長さも毛色も違う各プロダクトにエンジニアがどのように立ち向かっているのかが書かれている。  
本の内容の詳細は次の記事やポッドキャストを読めば把握できるのでここでの説明は省略する。



<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Ftechlog.voyagegroup.com%2Fentry%2Fengineers-in-voyage" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>
<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Ft-wada.hatenablog.jp%2Fentry%2Fvoyagebook" title="『Engineers in VOYAGE ― 事業をエンジニアリングする技術者たち』ができるまで #voyagebook - t-wadaのブログ" class="embed-card embed-blogcard" scrolling="no" frameborder="0" style="display: block; width: 100%; height: 190px; max-width: 500px; margin: 10px 0px;"></iframe>
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://ajito.fm/58/" data-iframely-url="//cdn.iframe.ly/naxUkm3"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# なぜ読もうと思ったのか
読む前の私のVOYAGE GROUPの認識は`はじめに`に記載されている当初の対象読者と言ったところだ。

> 本書のもともとの意図は「VOYAGE GROUPの名前は知っているものの、どんなエンジニアがどんなシステムを作っているかまでは知らなかった」という方に興味を持って手に取って読んでもらうことでした。

自分が働いている、働いていた会社以外ではどんなふうに開発を進めているのだろう？エンジニアリングをしているのだろう？というのは社会人ならば気になるのではないか。  
VOYAGE GROUPの認識は広告系の会社で、[ものすごいVim捌きの人](https://twitter.com/suzu_v)がいるらしい、夏のTreasureインターンの内容がすごいらしい（私がもし学生だったらなあ）くらいの認識だった。  
実際にVOYAGE GROUPのエンジニアの方を何人かフォローしているし、Twitterなどで界隈から良い評判が聞こえていたので読んでみることにした。

# わかったこと
まったくのゼロからプロダクトを作るスタートアップで働いている人はほとんどいないと思う。
一定の歴史のある会社では働いているならば新規プロダクトでも何かしら既存の社内プロダクトや基盤との連携に向き合う必要がある。  
本著は「新しい技術を使ってどうやったプロダクトを作っていくか」という話の比率よりも
**「如何に過去の技術的負債に立ち向かっていくか」という視点の話が多かった**と思う。  

## 避けられない負債との戦いにどう立ち向かうか
そのような避けては通れないレガシー、技術的負債と戦った話が記載されているのが本著だ。
たとえば次の引用のような「ああ、いつの間にかそういうこと起きてるよねえ…」みたいな課題に対してどう解決していったのか、当事者たちの言葉で聞ける。

> ね: アプリケーションのコードであれば、問題がある箇所を少しずつ直していけます。 しかしデータベースだと、たとえば「同じものを表していそうな要素があちこちにある」といった厄介な問題に悩まされます。同じような名前で、同じように使われていそうなカラムが、こっちのテーブルにもあっちのテーブルにもあるとき、必要なデータがどちらに入っているかを手探りで確かめるしかありません。
> 
> 150 第5章 サポーターズ ―― 事業の成長を止めない手段としてのシステム刷新

VOYAGE GROUP各社の話なので、各章のトピックとなるプロダクトが違えば開発体制も異なる。自分の体験や状況と近い話も見つけられるのではないか。
Web系企業には珍しそう（？）な「外部委託で作られた有識者不在のプロダクト」をどのようにリプレイスをかけていったという話もある。

## 良い意味で身も蓋もないことが書いてある
本著は口当たりのよいスマートな解答だけではなく、身も蓋もない真理が率直に書いてある。

> 自分を含め、それまでは誰もが「時間が無限にあればやりたい。けれど、いまはそれを忘れて緊急の対応を進めないといけないから、この部分のみはこう作っておきましょう」を繰り返していました。ところが、すずけんさんはそれを許容しないで、「この案件に取り組みたい。それには例外の統一化が必要だ。だからやっていく」と言ってやってしまった
>
> 31 第 1 章 fluct ――― 広告配信の舞台裏の技術者たち 

> ―― 関係ないところに手を出す力、放っておかない力、というのがまずあると思います。重要だけど緊急でないから誰も手をつけないところをゴリゴリ巻き取っていくためには、カッとなる力、放っておかない力が必要です。
> そのうえで、それを短期間で仕留める力ももちろん必要になります。
> 率直に言うと、技術的負債を返済できる企業とできない企業があるのは、この腕力の有無によるという面も否定できません。
> 
> 32 第 1 章 fluct ――― 広告配信の舞台裏の技術者たち 

一方、単純に技術力だけでは解決できない麺もある。**業務で問題を解決するときは「なぜ今それをやるのか」「なぜ新機能開発を時間を減らしてまでやるのか」というような組織視点**も重要になる。
このような組織としてどう立ち向かっているか、立ち向かったかという話も書かれていた。

> 福:率直なところ、葬りによる生産性の向上と、新機能の開発や収益増のためのシステム改善による生産性の向上は、単純に天秤にはかけられません。どちらを先にやるか、一概に結論を出すのはとても困難です。なので、駆け引き抜きで責任者どうしが互い に調整できる施策をそのつど選んで進めている感じです。
>
> ―― 責任者というのは、開発チームの責任者ですか?
>
> 福: いや、開発側のトップとビジネス側のトップです。具体的には開発本部長とメディア本部長になります。
>
> 106 第 3 章 VOYAGE MARKETING ― 20 年級大規模レガシーシステムとの戦い

Biz/Dev双方のトップの決断で技術的負債に立ち向かうのは最適解であれなかなかできないことだなと思った。

## 言われれば当たり前だけれど、普段できていないようなことへの示唆も多く含まれている。
**稼働するシステムに必要な視点、実際に負債を返却するときに必要な課題のブレイクダウン方法**などもふんだんに言語化されていた。

> ロギングすらされていないと気づきようがないので、それをメトリクスに入れるようにパッチを書いたりします。と同時に、障害が起こるまでわからないとも言っていられないので、起こるであろう障害を予想して先にメト リクスを取れるようにする、というのも意識的にやっています。
> 
> 36 第 1 章 fluct ――― 広告配信の舞台裏の技術者たち 

> ―― もちろん世の中には魔窟と呼ばれるような手がつけられないレガシーシステムがいくらでもありますが、それにしても 1200 はかなり大きな部類です。もはや頭に入るサイズを 超えてしまっているので、手をつけるのを諦めて結果的にさらにテーブルが増えていく、と いうレベルです。何らかのシステムを引き継いだとして、それだけの数のテーブルがあるこ とが判明したら、その時点で頭が真っ白になりますよね。
>
> 福: いまだに覚えていますが、周囲のエンジニアにこの事実を中間報告した際には、 まさにそういう反応がありました。この規模だと、そもそも資料や一覧を真面目に読 んでくれなくなります。これらを踏まえて、コツコツ改善し続けていく方向に舵を切 ろうと考えました。
>
> 86 第 3 章 VOYAGE MARKETING ― 20 年級大規模レガシーシステムとの戦い

各プロダクトで発生していた**問題に対して組織としてどのような戦略をとるのか、なぜそのように考えたのか**も多く述べられおり、自分の意志選択でもこのような視点を取り入れていきたいと思った。

> ランニングコストは、開発にばかりフォーカスしていると小さい話に見えるんですが、システムやサービスを長く運用するうえでは無視できない要素です。不必要なコストがかかっている状態では、どうしても優れた投資ができなくなります。新規開発や、よりベネフィットが高い決断、たとえばライターさんを増やすといった決断にあたっては、システムの定常的なコストを抑えていたことが大きかったと思います。
> 余計なランニングコストは、システムがもたらしている制約です。この制約が大きいほど、組織がとれる戦略が硬直化してしまいます。
> 
> 141 第 4 章 VOYAGE Lighthouse Studio ― 数十万記事のメディアをゼロから立ち上げる

**随所に挟まれる補足は1ページ以上の具体的な説明も多く、補足を読むだけでも問題解決のための知識、視点が学べた。**

## インタビューアの掘り下げが良い。
インタビュー記事や事例を読んでいて
**「よさそうだけれど、○○の視点を無視していない？」とツッコミを入れたくなる**ときがある。
本著でも尖った施策や方針が紹介されており「ん？それダイジョブなの？」みたいに思ったところがいくつかある。

> 河: ほかの多くのシステム開発と違う点があるとしたら、Zucksにはドキュメントがないことだと思います。そんなにおかしな話ではないと思っているんですが...。
> というのも、ぼくらエンジニアがやらないといけないのは、フルサイクルの流れを回すことなんです。一方で、一回でもドキュメントを書いてしまうと、そのドキュメントをメンテナンスし続けるという作業が発生してしまいます。さもないと、新しい人が入ってきたとき、古いドキュメントを有効な情報だと思って参照してしまうことになるでしょう。
> それを避ける目的もあって、よそではドキュメント化されるような情報でも、すべ てGitHubのissueだけで管理しています。
>
> 65 第 2 章 Zucks ― フルサイクル開発者の文化

そのような話では[@t_wada][t_wada]さんがインタビューアとして深堀りをしてくれているので、読後に「そうは言っても実際はどうなんだろうなあ」というようなモヤモヤがほとんどなく読み切れた。

> ―― いわゆる「ドキュメント」を書かずにすべてissueに書くという方法は、コードレベルであればうまく回りそうに思える一方で、全体の俯瞰とかアーキテクチャのようなものは示しにくいように思えます。
> そうした情報の伝達や共有はどのように解決しているのでしょうか?
>
> 66 第 2 章 Zucks ― フルサイクル開発者の文化


# 所感・今後どう活かすのか
本著にはVOYAGE GROUPでは何を基準としてどんな開発をしているのか、どんな選択をしているのかがふんだんに書かれていた。 
「あの会社はどんな働き方をしているんだろう？どんな風に課題を解決しているのだろう？」という興味を満たすには十分な内容だった。  
アドテク開発だけでなく、ウェブメディアや就活ポータルサイトの開発の話もある。Webエンジニアの話ばかりではなく、データサイエンスエンジニアの話もある。  
自分の経験に近い、同じようなシチューエーションにどう立ち向かったのか学べる章もあれば、「XXXの人たちはこういうふうに考えて働いているんだ」と知らない世界を学べる章もあった。
このように正しくも愚直に開発をしている会社、そこで働くプロフェッショナルの話を読むことができてよかった。自らの立ち振るまいにも取り入れていきたい。  
このような、中で働いている人たちが自分の会社を語っていく本をもっと読んでみたいとも思った。

# 参考
- [Engineers in VOYAGE ― 事業をエンジニアリングする技術者たち（紙書籍＋電子書籍）](https://www.lambdanote.com/products/engineers-in-voyage)
- [『Engineers in VOYAGE ― 事業をエンジニアリングする技術者たち』ができるまで #voyagebook | t-wadaのブログ](https://t-wada.hatenablog.jp/entry/voyagebook)
- [書籍「Engineers in VOYAGE 事業をエンジニアリングする技術者たち」が発売 #voyagebook | VOYAGE GROUP techlog](https://techlog.voyagegroup.com/entry/engineers-in-voyage)
- [ajitofm 58: Engineers in VOYAGE 出版しました #voyagebook](https://ajito.fm/58/)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4908686092&linkId=019197b9d03c1b3dd7c1575c107f24dc"></iframe>

[t_wada]: https://twitter.com/t_wada