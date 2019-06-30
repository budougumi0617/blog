+++
title= "[書評] UNIXという考え方―その設計思想と哲学 を読んで「単純さ」の大切さを改めて学んだ"
date= 2019-06-30T20:35:18+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["review", "unix"]
tags = ["review", "unix", "book"]
keywords = ["UNIXという考え方", "書評"]
twitterImage = "2019/06/30_bgp.png"
+++


「UNIXという考え方―その設計思想と哲学」を読んだのでその感想をまとめる。


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4274064069&linkId=395b62f54cc9dfae0b7fe848d5eb0914"></iframe>

<!--more-->

# 所感
古い本（2001年初版発行）だが、特定の実装の解説などではなく哲学の話なので時代遅れな内容を感じなかった。  
むしろリーンスタートアップなどに通じるスモールスタートの考えや、最近のマイクロサービス志向に通じる責務の分担の考えなど、不変的な重要な哲学が非常にわかりやすくまとめられていた。  
索引を入れても150ページにも満たないし、私はほとんどの内容に共感できたのでサクサクと読むことができた。


# どんな本なのか
この本は以下の章で成り立っている。日本語版監訳者のページで触れられているとおり、UNIXが作られる中で確立された設計哲学・方法論を学ぶことができる。  
その哲学は特定のプログラミング言語やUNIX OSに依存した内容ではないので、ソフトウェアエンジニアならば誰が読んでも参考になるだろう。

- 第1章 UNIXの考え方
- 第2章 人類にとって小さな一歩
- 第3章 楽しみと実益をかねた早めの試作
- 第4章 移植性の優先順位
- 第5章 これこそ梃子の効果！
- 第6章 対話的プログラムの危険性
- 第7章 さらなる10のUNIXの考え方
- 第8章 一つのことをうまくやろう
- 第9章 UNIXとその他のオペレーティングシステムの考え方

# なぜ読もうと思ったのか
"Do One Thing and Do It Well"（ひとつのことをうまくやれ）という格言は知っていたが、それがどのような背景で出来たのか知りたかった。  
また、私が好きなGoの言語哲学とUNIXの哲学が似ていそうなのも大きかった。GoはRob Pike氏らが作ったプログラミング言語で単純さ（`Simplicity`）に重きを置いた言語である。

- Simplicity is Complicated
  - https://talks.golang.org/2015/simplicity-is-complicated.slide

Goの実装や設計を考える中でもUNIXの考え方が活かせるのでないかと考えた。また、kaneshinさんの著書でもUNIXの考え方とGoの親和性が言及されており、一度ちゃんと読みたいと思っていた。

- Go言語らしくGoコードを実装するための手法と思想
  - https://kaneshin.booth.pm/items/1061168

他にも、マイクロサービスアーキテクチャは単一機能を独立したマイクロサービスに分けてシステムを構築していく設計アプローチだ。
各マイクロサービスはまさに「ひとつのことをうまくやれ」になる必要がある。  
また、もっとコードベースで言うと、パッケージ（あるいはnamespaceや”層”）を考える上で、「あるひとつのこと」を担うパッケージ構成はどのように切り分けていくのか？という模索をつねに考えている。
このような設計を考えるときでも「単純さ」の考えが役に立つのではないかと思った。


# 読んでわかったこと
全体を通してシンプルにつくること、シンプルを積み上げることシステムを構成することで得られるメリットを学ぶことができた。  
とくに良いなと思ったところを挙げておく。

## 第3章 楽しみと実益をかねた早めの試作
> 試作が早ければ速いほど、製品のリリースは早まる。試作によって何がうまくいくかが分かり、さらに重要なことに何がうまくいかないかがわかる。

私は恥ずかしながらリーン・スタートアップを未読なので概要しか知らないが、UNIXの考え方の時点でリーンやアジャイルのスタートアップの「まず小さく動くものをつくる」という精神が言及されていることに驚いた。  
たしかに考えてみれば、OSのような汎用性や拡張性も考慮しなければけない相当規模のシステムを構築するに詳細までは机上で設計していくのは不可能に近いだろう。  
早い試作をし、仮定と検証を繰り返すことで精度を高めていくアプローチもUNIX哲学の1つだと知れたのは意外な収穫だった。

## 第5章 これこそ梃子の効果！
「5.2 定理7: シェルスクリプトを使うことで梃子の効果と移植性を高める」では6つのUNIXコマンドを組み合わせたワンライナーが紹介されている。
（あるUNIXのバージョンでは）この6つのUNIXコマンドの実装コードの合計は約1万行になるらしい。1行のワンライナーで1万行分のコードを実行できる強力さが強調されている。  
6つのコマンドの組み合わせは任意の組み合わせであり、この6つのコマンドの組み合わせと同じ結果を得る1つのコマンドを作成することもできる。
しかし、1つのコマンドにしてしまうと、間に自作のコマンドを挟んだり、入出力を別に変えたりすることが難しくなるだろう。
「ひとつのことをうまくやる」複数のコマンドに分かれているからこそ、シェルスクリプトやワンライナーコマンドは柔軟な状況に対応できることを改めて気付かされた。

## 第6章 対話的プログラムの危険性
本書ではシンプルなインターフェースがなぜ良いのかという説明が随所に書かれている。  
例えば、「6.1 定理8: 過度な対話的インターフェースを避ける」では以下の説明がされていた。

- システムの操作インターフェースを把握していないといけない
- 人間が利用することが前提となるめ、他のプログラムと結合するのが難しい

これはプログラミングでも似たようなことが言えると思う。  
特定のクラスや構造体へ入出力が依存しているコードをラップするコードを書いたりした経験は誰しもあるのだろうか。  
インターフェースを定義したり、特定の入出力構造に依存しないように設計・実装することで柔軟なライブラリ・パッケージを作ることができる。


# 今後どう活かすのか
実際に自分の書くコード、考える設計にUNIXの考えを適応させていきたい。  
ひとまず、今作っている趣味コマンドをUNIX的に考えてみた。

## UNIXという考え方的にコマンドを作ってみる

- https://github.com/budougumi0617/hc

作成中のこのリポジトリの`hc`コマンドは、あるRSSフィードの中にある各記事のはてなブックマークを一覧表示する予定のコマンドだった。

最初はコマンドの処理内容を以下のように考えていた。

- コマンドの引数はブログのRSSフィードのURLとする
- コマンドの中でRSSフィードを取得・パースし、各記事のURLの一覧を作成する
- 各記事のURとそれぞれのはてなブックマーク数を表示する。

イメージとしてはこんな使い方だ。

```bash
$ hc https://budougumi0617.github.io/index.xml
  831   https://budougumi0617.github.io/2019/03/03/review-output-encyclopedia/
  268   https://budougumi0617.github.io/2019/05/12/pass-aws-solution-architect-associate/
  191   https://budougumi0617.github.io/2019/01/16/develop-google-apps-script-by-typescript/
  ...
```

## シンプルな入出力にしてみる

「ひとつのことをうまくやる」ということを考えた時、このコマンドでやりたかったことは「複数渡されたURLのそれぞれのはてなブックマーク数を表示する」だけだったのではないかと考えた。  
そこで、標準入力から渡されたURLのはてなブックマーク数を取得するだけを実装してみた。たとえば、別のコマンドの出力からURLを取得してコマンドに入力してみる。

```bash
$ echo "http://hoge.gom/\nhttps://google.com/" | ./hc
    0   http://hoge.gom/
 1272   https://google.com/
```

URLの一覧を保存したファイルから自作コマンドに入力してみる。

```bash
$ cat urls.txt
https://budougumi0617.github.io/2019/06/20/golangtokyo25-read-again-awesome-go-article/
https://budougumi0617.github.io/2019/06/13/explore-go-specification-and-implementation/
https://budougumi0617.github.io/2019/03/03/review-output-encyclopedia/
https://budougumi0617.github.io/2019/05/12/pass-aws-solution-architect-associate/
https://budougumi0617.github.io/2019/01/16/develop-google-apps-script-by-typescript/

$ cat urls.txt | ./hc
    0   https://budougumi0617.github.io/2019/06/20/golangtokyo25-read-again-awesome-go-article/
   21   https://budougumi0617.github.io/2019/06/13/explore-go-specification-and-implementation/
  831   https://budougumi0617.github.io/2019/03/03/review-output-encyclopedia/
  268   https://budougumi0617.github.io/2019/05/12/pass-aws-solution-architect-associate/
  191   https://budougumi0617.github.io/2019/01/16/develop-google-apps-script-by-typescript/
```

# 別のコマンドに処理を移譲する

シンプルな出力フォーマットにしておくことで、コマンドの出力を他のコマンドの入力にもできる。  
はてなブックマーク数で出力結果をソートする機能もつけてみようと思ったが、それは`sort`コマンドに任せられるかと考えた。  
以下は`sort`コマンドを使って実行結果をはてなブックマーク数順でソートしてみる例だ。

```bash
$ cat urls.txt| ./hc | sort -k 1 -t$'\t' -nr
  831   https://budougumi0617.github.io/2019/03/03/review-output-encyclopedia/
  268   https://budougumi0617.github.io/2019/05/12/pass-aws-solution-architect-associate/
  191   https://budougumi0617.github.io/2019/01/16/develop-google-apps-script-by-typescript/
   21   https://budougumi0617.github.io/2019/06/13/explore-go-specification-and-implementation/
    0   https://budougumi0617.github.io/2019/06/20/golangtokyo25-read-again-awesome-go-article/
```

今回実装したコマンドは与えられたURLのはてなブックマーク数を取得する機能しかない。  
しかし、(MacOS上だが)コマンドラインのUNIXという考えが満たされた空間では、シンプルな機能の組み合わせで柔軟な結果が得られることが出来た。  
最初に考えていた引数にRSSフィードのURLを与える少し複雑な機能のままでは、このような柔軟な入力を受けることはできなかっただろう。  
中身のコードはまだハードコーディングされた標準入出力があったり非構造化状態なので、設計としてもシンプルさを心がけてリファクタリングしていくつもりだ。
（RSSフィードから記事のURL一覧を取得するコマンドは別に作ろうと思う。）

# 終わりに
今まで少し古いこと・（自宅の収納問題的に）電子書籍がないこと、を理由に読んでいなかった。  
今回はよしたく([@yoshitaku_jp](https://twitter.com/yoshitaku_jp))さんに頂いたので読み始めたのだが、もっと早く読めばよかったと後悔する非常によい内容だった。  
本の厚さ的にも2・3日程度で読める厚さだし、未読のソフトウェアエンジニアはぜひ読んだほうがいいと思う。


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4274064069&linkId=395b62f54cc9dfae0b7fe848d5eb0914"></iframe>

# 参考
- [Simplicity is Complicated](https://talks.golang.org/2015/simplicity-is-complicated.slide)
- [Go言語の父と呼ばれるRob Pike氏による基調講演～Go Conference 2014 Autumn基調講演1人目](https://gihyo.jp/news/report/01/GoCon2014Autumn/0001)
- [Go言語らしくGoコードを実装するための手法と思想](https://kaneshin.booth.pm/items/1061168)


