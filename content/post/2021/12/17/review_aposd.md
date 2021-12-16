+++
title= "[書評] A Philosophy of Software Design を読んだ。複雑性を理解し、戦う術を身につけた"
date= 2021-12-15T23:15:13+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["review"]
tags = ["book","design"]
keywords = ["aposd","A Philosophy of Software Design"]
twitterImage = "2021/12/17_aposd.jpeg"
+++

この記事は「おすすめ本 Advent Calendar 2021」17日目の記事となる。  

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://adventar.org/calendars/6382/embed" data-iframely-url="//cdn.iframe.ly/pOcNyR6"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

[今年読了した40冊ほどの本](https://booklog.jp/users/budougumi/stats)の中で一番よかった「A Philosophy of Software Design」を紹介する。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B09B8LFKQL&linkId=bd9397f5a902d0c2ba71700b09d6f547"></iframe>

<!--more-->

# 所感
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/EnterpriseQualityCoding/FizzBuzzEnterpriseEdition" data-iframely-url="//cdn.iframe.ly/ZobflD8?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# どんな本なのか
https://www.amazon.co.jp/dp/B09B8LFKQL

Amazonの紹介文を引用する。

> This book addresses the topic of software design: how to decompose complex software systems into modules (such as classes and methods) that can be implemented relatively independently. The book first introduces the fundamental problem in software design, which is managing complexity. It then discusses philosophical issues about how to approach the software design process and it presents a collection of design principles to apply during software design. The book also introduces a set of red flags that identify design problems. You can apply the ideas in this book to minimize the complexity of large software systems, so that you can write software more quickly and cheaply.

要約すると

本書は、ソフトウェア設計のテーマとして複雑なソフトウェアシステムをいかに分解するかということを取り上げており、
大規模なソフトウェアシステムの複雑さを最小限に抑え、より早く、より安くソフトウェアを書くための哲学、テクニックがまとめられている。

- 「複雑性の管理方法」の紹介
- ソフトウェア設計プロセスにどのようにアプローチするかという哲学的な問題
- ソフトウェア設計時に適用すべき設計原則のコレクション
- 設計上の問題を特定するためのレッドフラッグ（赤信号）の紹介

著者がGoogleで直接話した動画もある。
<div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/bmSAYlu0NcY?rel=0" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture;"></iframe></div>

なお、本著は2021年7月に2nd editionが発売されている。  
初版しか読んでいない人は新たな章が追加されているのでぜひ読んだほうが良い。  
（未購入者も）差分をここから読むことができる。

https://web.stanford.edu/~ouster/cgi-bin/book.php

# なぜ読もうと思ったのか
初版発売当初、尊敬するエンジニアのみなさんが相次いで絶賛していた。  

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://yshibata.blog.ss-blog.jp/2019-03-12" data-iframely-url="//cdn.iframe.ly/wdaa5uV?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fsyfm.hatenablog.com%2Fentry%2F2019%2F01%2F22%2F165339" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

- [2018年振り返り | Taichi Nakashima](https://deeeet.com/writing/2018/12/29/2018/)

ただ、翻訳版の出る見込みがないとのことで英語が苦手な自分は読むのを諦めていた。  
しかし、技術選定やアーキテクチャを考えるような機会も増え、もっと自分の技術的判断基準の拠り所をはっきりさせたいと思うようになった。  
そこでこの本に改めて挑戦し毎日10分ほど辞書サイトやKindleの単語検索を駆使しつつ少しずつ読むことにした。

# 読んでわかったこと
本書では終始「複雑性」の解説と戦い方が示されている。  
「なんとなく悪い感じ」という自分の感覚を定性的に表現できるようになった。  
なぜシンプルなインタフェースがよいのか？どうすればよいモジュールが設計できるのか？なにかの良し悪しを考えるときに「複雑さ」という判断基準を持てるようになった。  
また、コメントに対する意識の持ち方は非常に参考になった。  
経験上コメントに力を入れるエンジニアは総じて優秀だったので自分も本著の書き方を意識してもっとコメントを書いていこうと思った。

- 複雑さとは、ソフトウェアシステムの構造に関するもので、システムを理解し修正することを困難にするものである。
    - **複雑かどうか？を決めるのは書き手ではなく読み手**
- 複雑さは大別して3種類ある
    - Change amplification
    - Cognitive load
    - Unknown unknowns
- Deep Module
- EasyとSimple
- エラー（例外）について
- 1回でよい設計をすることはできない
- コメント無しでシグネチャに頼った書き方していると浅いコードになっていく
- 実装を読まないと理解出来ないメソッドは抽象化ができていない
    - 引数のシグネチャだけじゃ表現力がたりない
- コメントはちゃんと書くこと。
    - 「良いコードならコメントが不要」ということはない
- どの粒度のコメントを書くのか意識する。
    - Lower-level comments add precision
    - Higher-level comments enhance intuition
    - Interface documentation
    - Cross-module design decisions
- コメントを書くベストなタイミングは開発プロセスの最初に書くこと。コメントを書くことが設計プロセスの一部になる。
- どうやってメンテナブルなコードを書くか
- 一貫性
- 明瞭性
- 優れたソフトウェア設計のもっとも重要な要素のひとつは、重要なものとそうでないものを分けること

# 今後どう活かすのか
今は自分のチームでこの本の輪読会を行ない、チームで改めて複雑さや設計について議論している。  
第三者と本の中身や今までの経験について話すことでより咀嚼して自分のスキルとして身につけていきたい。

また、「はじめて英語の技術書を一冊ちゃんと読んだ」という点でも大きく自信をつけることができた。  
今はまた同じアプローチで少しずつ新しい[英語の本](https://www.amazon.co.jp/dp/B0923C9WB5)を読んでいる。  
元が取れそうになくて諦めていた[オライリーのサブスク](https://www.oreilly.com/online-learning/)に登録したり、技術書以外の英語の読書もできたらなと思う。  
英語アレルギー克服という意味でもこの本を今年読めたのはとてもよい経験になった。

# 参考
- [おすすめ本 Advent Calendar 2021](https://adventar.org/calendars/6382)
- [[書評] UNIXという考え方―その設計思想と哲学 を読んで「単純さ」の大切さを改めて学んだ](/2019/06/30/review-the-unix-philosophy/)
- [A Philosophy of Software Design を読んだ | blog.syfm](https://syfm.hatenablog.com/entry/2019/01/22/165339)
- [『A Philosophy of Software Design』：柴田 芳樹 (Yoshiki Shibata)](https://yshibata.blog.ss-blog.jp/2019-03-12)
- [2018年振り返り | Taichi Nakashima](https://deeeet.com/writing/2018/12/29/2018/)
- https://www.oreilly.com/online-learning/

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B09B8LFKQL&linkId=bd9397f5a902d0c2ba71700b09d6f547"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4274064069&linkId=1cf851a3fb12fa3da484fa000250000a"></iframe>