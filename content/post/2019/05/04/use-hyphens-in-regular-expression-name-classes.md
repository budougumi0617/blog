+++
title= "正規表現の文字クラス（角括弧[]）にハイフンを含みたいときは、ハイフンを最初または最後に書く"
date= 2019-05-04T07:41:57+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["grep", "CLI"]
tags = ["ERE", "grep", "CLI"]
keywords = ["正規表現", "Regular Expression", "grep"]
twitterImage = "twittercard.png"
+++

正規表現をいじっていたらハマったのでメモ。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873113598&linkId=59ba8fe3cd580af67049dcacf9c16e40"></iframe>

<!--more-->

# TL;DR
- 文字クラス(`[]`)でハイフン（`-`）を指定しているつもりがちゃんと判定されない
- 文字クラスの中でハイフンを指定するときは最初か最後にハイフンを置かないと範囲指定あつかいされてしまう。

# 文字クラス(`[]`)でハイフン（`-`）を指定しているつもりがちゃんと判定されない
`Makefile`の自己文書化をしていた。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://postd.cc/auto-documented-makefile/" data-iframely-url="//cdn.iframe.ly/6P77kjy?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

上述のリンク先では以下のようなワンライナーが紹介されている。
このワンライナーでは、序盤の`@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST)`部分で`Makefile`内の対象行を拾って後続の処理を行なっている。

```bash
.PHONY: help

help:
    @grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
```

私の`Makefile`には`make docker.run`などのようなドット（`.`）を含むコマンド定義があったが、この正規表現にドットは含まれていない。  
そのため、上記の正規表現中の`^[a-zA-Z_-]`の部分を`^[a-zA-Z_-\.]`と変更したところ、ハイフンを含む行がヒットしなくなってしまった。

# 文字クラスの中でハイフンを指定するときは最初か最後にハイフンを置かないと範囲指定扱いされてしまう。
`\-`とエスケープをして`^[a-zA-Z_\-\.]`のように書いても当然意味がなかった。結局検索して解決することができた。一次ソース（仕様）は見つけられなかった。（正規表現の仕様はWeb上には公開されていないのだろうか？）  

- 正規表現　-ハイフン　判定
  - http://origin8.info/blog/?p=490

節タイトルの通り、文字クラスの中でハイフン（`-`）を指定するときは最初か最後に指定しないといけないようだ。  
`^[a-zA-Z_-\.`を`^[a-zA-Z_.\-]`と書きかえることで、ハイフンを含んだ文も、ドットを含んだ文もgrepすることができる正規表現を書くことができた。


# 終わりに
正規表現は記号が検索しにくいというところもあってなかなか調べるのが難しい（`[]`で囲う表記を文字クラスと呼ぶことも今回初めて知った）。
オライリー本を買ってちゃんと読むべきか。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4873113598&linkId=59ba8fe3cd580af67049dcacf9c16e40"></iframe>

# 参考
- [正規表現　-ハイフン　判定](http://origin8.info/blog/?p=490)
- [正規表現言語 - クイック リファレンス | Microsoft Docs](https://docs.microsoft.com/ja-jp/dotnet/standard/base-types/regular-expression-language-quick-reference)
  - `.NET`用のリファレンスのようだが、初歩的なところがちょうどいい文量でまとめてあったので。
- [RegExr](https://regexr.com/)
  - オンラインで正規表現を確認できる。正規表現で検索したい例文を自分で用意することもできたので便利。
