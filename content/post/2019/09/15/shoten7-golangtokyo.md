+++
title= "golang.tokyoの技術書典7新刊「Gopherの休日2019秋」に寄稿しました #技術書典"
date= 2019-09-15T13:04:11+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["Go", "golangtokyo", "技術書典"]
keywords = ["技術書典", "技術書典7", "golang.tokyo", "golangtokyo", "Go", "golang", "golang.org/x"]
twitterImage = "2019/09/15_shoten7.png"
+++

2019/09/22に行われる技術書典7へ、golang.tokyoも参加します。  
私は、今回の新刊である「Gopherの休日2019秋」に「準標準パッケージ（`golang.org/x`）の早めぐり」という内容で寄稿しました。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://techbookfest.org/event/tbf07/circle/5174941137764352" data-iframely-url="//cdn.iframe.ly/yAYopWf"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# golang.tokyo 技術書典7 新刊「Gopherの休日2019秋」について
今回の新刊は私以外にも5名の方々が執筆しており、以下のような内容になっています（敬称略）。

- 準標準パッケージ（golang.org/x）の早めぐり / [@budougumi0617][budougumi0617]
- Goとベイズ理論でシンプルな記事分類を実装してみよう！ / [@po3rin][po3rin]
- GUIライブラリGioで遊んでみよう / [@knsh14][knsh14]
- gomobileの現況解説 / [@hajimehoshi][hajimehoshi]
- Secure Coding Practices in Go / [@kaneshin0120][kaneshin]
  - [\[技術書典7\] golang.tokyoへ寄稿しました by kaneshin | Medium][kaneshin_blog]
- Goの新しいコントラクト / [@tenntenn][tenntenn]
  - [技術書典7においてGOLANG.TOKYOのサークルに寄稿しました | tenntenn.dev][tenntenn_blog]

前回同様、すべての内容が技術書典7のための書き下ろしになっており、
機械学習、セキュアコーディングからGo2のコントラクト、gomobile、GUIフレームワークまで多種多様です。
価格は計108ページで販売価格1,000円となります。 （なお、売上はすべてgolang.tokyoのノベルティ作成などに使われます）  
また、今回は[@tottie_designer][tottie]さんに表紙・裏表紙を作成していただき、製本も行ないました。
製本数はあまり多くありませんが、表紙・裏表紙もとても素晴しいのでぜひ製本版を手にとっていただけると幸いです。

[budougumi0617]: https://twitter.com/budougumi0617
[knsh14]: https://twitter.com/knsh14
[po3rin]: https://twitter.com/po3rin
[hajimehoshi]: https://twitter.com/hajimehoshi
[kaneshin]: https://twitter.com/tottie_designer
[tenntenn]: https://twitter.com/tenntenn
[tottie]: https://twitter.com/tottie_designer

# 寄稿内容について
私が寄稿した「準標準パッケージ（`golang.org/x`）の早めぐり」では、2019年9月現在`golang.org/x`パッケージとして配信されている準標準パッケージを一覧しています。  
以下、寄稿した文書から内容紹介の節を引用しておきます。

## 概要・ゴール
本章では、Goの準標準パッケージとして配信されているリポジトリにどんなものリポジトリがあるのかを紹介します。
リポジトリの中にはGoの今後のバージョンのために試験的に作成されたパッケージから、実務での利用に耐えうるパッケージまでさまざまなコードが含まれています。
中には実際に実務でGoを開発する際にほぼ確実に利用するであろうコマンドラインツールも含まれています。
このような準標準パッケージを学習することで、本章では次のようなことを学ぶことをゴールとします。  

 * 準標準パッケージで提供される各種ツールを知ることで日々の開発の生産性を上げる
 * 準標準パッケージで提供されるコードを利用することでコーディングの品質を上げる（車輪の再発明を避ける）
 * 今後リリースされるGoのバージョンに取り込まれるであろう、新しいGoの仕組みを先取りして学習する

まず、「`golang.org/x`配下にあるパッケージとは」では`golang.org/x`配下にあるパッケージの位置付けを簡単に述べます。
「`golang.org/x`で提供されているパッケージ一覧」では、`golang.org/x`配下にあるすべてのパッケージの簡単な紹介を述べます。
最後に「`golang.org/x`で提供されているパッケージを使ったコーディング」で実践的な実装を含んだパッケージの詳細とサンプルコードを記載します。


# おわりに
当日は販売も担当します。「技術書典7 か12C golang.tokyo」でお待ちしております。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">技術書典7用の原稿「Gopherの休日 2019秋」を先ほど入稿しました。今回は初めて紙媒体で販売します。<br>表紙・裏表紙は <a href="https://twitter.com/tottie_designer?ref_src=twsrc%5Etfw">@tottie_designer</a> さんに描いていただきました。<br>Goに関するアレコレを6名で寄稿し合計116ページになっています！当日会場でぜひご覧ください！ <a href="https://twitter.com/hashtag/%E6%8A%80%E8%A1%93%E6%9B%B8%E5%85%B8?src=hash&amp;ref_src=twsrc%5Etfw">#技術書典</a><a href="https://t.co/dHqC7waNlk">https://t.co/dHqC7waNlk</a> <a href="https://t.co/0kJxhYBUmT">pic.twitter.com/0kJxhYBUmT</a></p>&mdash; golang.tokyo 9/22 技術書典7 3F か12C (@golangtokyo) <a href="https://twitter.com/golangtokyo/status/1171657235652653056?ref_src=twsrc%5Etfw">September 11, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 関連情報
- [技術書典7 golang.tokyoサークル詳細](https://techbookfest.org/event/tbf07/circle/5174941137764352)
- [技術書典7においてGOLANG.TOKYOのサークルに寄稿しました | tenntenn.dev][tenntenn_blog]
- [\[技術書典7\] golang.tokyoへ寄稿しました by kaneshin | Medium][kaneshin_blog]

[tenntenn_blog]: https://tenntenn.dev/ja/posts/shoten7-golangtokyo/
[kaneshin_blog]: https://medium.com/@kaneshin0120/%E6%8A%80%E8%A1%93%E6%9B%B8%E5%85%B87-golang-tokyo%E3%81%AB%E5%AF%84%E7%A8%BF%E3%81%97%E3%81%BE%E3%81%97%E3%81%9F-e6197d7e12ac
