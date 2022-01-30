+++
title= "[Notion] Mac Appでフォントのサイズが大きくならない"
date= 2022-01-30T16:20:47+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["notion"]
tags = ["notion"]
keywords = ["notion","Mac App","フォントサイズ","大きくできない"]
twitterImage = "logos/notion.jpeg"
+++
NotionのMac Appでアプリ全体のフォントサイズを大きくできなかったのでメモ。

<!--more-->

# TL;DR
- NotionのMac Appでフォントサイズが大きくできなかった
    - 公式のショートカットに載っている`Cmd` + `+`で大きくならない
- 人によって設定が異なる？`Cmd` + `+`でフォントサイズを変えられている同僚もいた

私の環境

- M1 Macbook Pro(14インチ、2021) USキーボード
- HHKB USを利用
- macOS Monterey 12.1

# Notion Mac Appでフォントサイズが大きくできない

Notionを使っていて、PCで使うときはブラウザとMac Appを併用している。  
間違えて`Cmd` + `-`を押してしまってフォントサイズが小さくなってしまった。感覚的に`Cmd` + `+`を押下してみたがフォントサイズは変わらなかった。


# 公式ヘルプをみてもわからない
Notionのショートカットキーのヘルプページはこれ。ここには`Cmd` + `+`と書いてある。

https://www.notion.so/help/keyboard-shortcuts
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.notion.so/help/keyboard-shortcuts" data-iframely-url="//iframely.net/00XBwZR?card=small"></a></div></div><script async src="//iframely.net/embed.js" charset="utf-8"></script>

`Cmd` + `^`と書いている人もいたがこれでもダメだった。

- 【Notion】文字サイズの変更方法！
    - https://aketama.work/notion-text-size

# `Cmd` + `=` でフォントサイズを大きくできた
いろいろ試し打ちしてみた結果、`Cmd` + `=`ならフォントサイズを大きくできた。  
同僚は`Cmd` + `+`でフォントサイズを大きくできるらしい。Macのショートカットキーや他のアプリとショートカットが競合してるわけでもなく理由はわからない。

# 終わりに
その他にも`Cmd` + `[`で前のページに戻れず、`Cmd` + `←`で戻ったりしている。  
文書を書くときに苦労していないのでキーマップがおかしいことはないと思うのだけれども…。
Notionはメニューバーに操作がほとんどでないのでそこからショートカットキーを確認することもできない。アプリデザインについても考える機会になった。

# 参考
- https://www.notion.so/help/keyboard-shortcuts
- [【Notion】文字サイズの変更方法！](https://aketama.work/notion-text-size)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B09JS3D9C6&linkId=07cb87e0992fd01e436eb067fe1bc4a9"></iframe>