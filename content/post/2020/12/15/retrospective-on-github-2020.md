+++
title= "2020年振り返り(GitHub編)"
date= 2020-12-15T09:20:57+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["esssay"]
tags = ["Retrospective", "github"]
keywords = ["振り返り", "GitHub"]
twitterImage = "twittercard.png"
+++

今年も例年通りGitHubの活動を振り返った。

この記事は、[write-blog-every-week Advent Calendar 2020](https://adventar.org/calendars/5530)の15日目の記事になる。  
14日目は[@tada_infra](https://twitter.com/tada_infra)さんの[毎週ブログを書くことを3年間続けてみた1年を振り返る](https://sadayoshi-tada.hatenablog.com/entry/2020/12/14/090000)だった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://adventar.org/calendars/5530/embed" data-iframely-url="//cdn.iframe.ly/CFIQIEi"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# 2020年のコントリビュート
ドキュメントの修正を2件程度でびっくりするほど何もしてなくて凹んだ…。

- hashicorp/terraform-website
    - https://github.com/hashicorp/terraform-website/pull/1554
- h-michael/lsm
    - https://github.com/h-michael/lsm/pull/5
- DataDog/documentation
    - https://github.com/DataDog/documentation/pull/7472

あとはPixelaにAPI追加のリクエストissueを作ったくらいだった。
- https://github.com/a-know/Pixela/issues/15

うーん、触ったOSSは増えているのにな。来年はもっとOSS活動を増やしていきたい。

# OSS
今年は3つOSSを作った。わりかし実用的なものを作れたので満足している。  
PRをいただけたりもして作ってよかったなと思う。  
来年はプロダクトで使えるような便利ツールを作ってみたい。AWS SDKをラップする何かかテストツールかな？
## leetgode
- https://github.com/budougumi0617/leetgode
    - LeetCode CLI for Gophers. This CLI can generate a skeleton code with test code.
    - [LeetCodeをローカルで効率よく解くためのGo専用CLIを作った](/2020/10/06/release_leetgode/)

leetcodeをやるのに既存のツールでは不満だったため。  
ただこのツールもHTMLパーサを作ってもうちょっと改良したほうが良さそう。GraphQLに触れたりポーリング実装が必要だったり思ったより利用技術は幅広かった。

## regexponce
- https://github.com/budougumi0617/regexponce
    - Analyzer: Verify that regular expression compilation just called at once.
    - [[Go] パフォーマンスが悪い正規表現パッケージの使い方をチェックするlinterを作った](/2020/08/20/regexponce/)

手習いで静的解析ツールを作ったことはあったものの、実用に耐えうるものは作ったことがなかったため。  
静的解解析ツールはもっとカジュアルにすぐ作れるようになりたい。

## terraform-provider-pixela
- https://github.com/budougumi0617/terraform-provider-pixela
    - Terraform Provider for Pixela( https://pixe.la/ )
    - [TerraformでPixelaのグラフを宣言できるProviderを作った #pixela](https://budougumi0617.github.io/2020/12/11/terraform_provider_pixela/)

雑誌の連載でTerraform Providerの書き方を知ったので書いてみた。  
terraform-provider-awsなどのテストも以前より読めるようになったのでこれはかなりやった甲斐があった。  
来年はこの素振りを生かしてhashicorpのproviderにコミットしたい。

## その他1
- Add CI settings: tsc build, and eslint check
    - https://github.com/budougumi0617/blog-kpi-collector/pull/17

昨年作った個人ブログのKPI計算claspプロジェクトのTS化を完了した。

## その他2
private repoでGo+ECSみたいな素振りアプリを作りはじめた。まだ自動CD設定してAWS上で動きはじめたくらい。  
NewrelicなどのSaaSを使いこなすための素振りをしたい（個人AWSの料金がそこそこ発生しているのでちゃんと有効活用しないと）。

# Dotfile
https://github.com/budougumi0617/dotfiles

 - 2020年の差分
  - https://github.com/budougumi0617/dotfiles/compare/3656b14...ff8d1e
    - [vim-goを使わず、LSP（gopls）を使ってVimのGo開発環境を構築する](/2020/07/24/make_vimrc_with_lsp/)

vim-goを捨ててLSPに移行した。それにあたりvimrcを一度全部捨てた。

# その他今年作ったリポジトリ
[created:>=2020-01-01 is:publicで絞った結果](https://github.com/budougumi0617?utf8=%E2%9C%93&tab=repositories&q=created%3A%3E%3D2020-01-01+is%3Apublic&type=&language=)

## leetcode
- https://github.com/budougumi0617/leetcode

素振りがてら競プロの過去問を解いている。leetcodeは入出力を関数として考えられるので好き。
## autorelease-by-release-it-on-actions
- https://github.com/budougumi0617/autorelease-by-release-it-on-actions

社のブログに寄稿した際のサンプルリポジトリ。  

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fautomatic-release-on-github-actions" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

最近はActionsでCI/CDを書くことが多いのでだいぶ慣れてきた。  
Actionsは近々ワークフロー中に承認を挟むこともできるようになるらしいので、来年も継続的に学習が必要そうだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.blog/2020-12-08-new-from-universe-2020-dark-mode-github-sponsors-for-companies-and-more/" data-iframely-url="//cdn.iframe.ly/cUGRT2v?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## cimg_go
- https://github.com/budougumi0617/cimg_go
- [[Go]次世代イメージcimg/goとcircleci/go Orbsを使った2020年版CircleCIの環境構築](/2020/06/08/circleci_cimg_go_2020/)

CircleCIのCI用イメージが次世代化していたので書いた。  
6月時点でpublic repoを検索してみたときはほとんど使っている人がいなかったので、いい記事が書けたかなと思っている。

## budougumi0617
- https://github.com/budougumi0617/budougumi0617

プロフィールページにREADMEを置けるやつ。

# Activity
2020年のコミット履歴。

https://github.com/budougumi0617/

## Publicなコミット = 1,063コミット
![Public only](/2020/12/glass-only-public-2020.png)


- 2017年 1,051コミット
- 2018年 1,286コミット
- 2019年 1,357コミット

今年はいろいろ[^1]あって4か月休止期間があったのだが、そのわりにはコミットしていたびっくりした。
つまり来年はもっと頑張れるかな？

[^1]: 来週の全体振り返りに書く


## Public + Privateなコミット = 3,584コミット
![Public and Private](/2020/12/glass-all-2020.png)

- 2017年 1,801
- 2018年 2,192
- 2019年 3,095

前述のとおりだったが結構コミットしていた。
（コーディングだけが仕事ではないが、）もっと事業に貢献できるようにコードはガンガン書いていきたい。

--- 

# まとめ（KPT）
KPTを書く前に[去年のKPTのTry](/2019/12/14/retrospective-on-github-2019/)は以下だった。

## 去年のTry
- 業務で貢献してprivateコミットを増やす
- 3月までにGAEで家族用のLINE Botを作る
- 半年に1つ以上2桁StarがもらえるようなOSSを公開する
- Vue.jsとFirebaseを使ってフロントエンドを構築してみる
- 定期的にTryの進捗を確認する
- OSSに対して機能追加のPRを6個以上出す

privateなコミットはそこそこ増やせた。LINE BOTも使いはしなかったがひとまず実装はできた。  
ただ、他がまったくできていない…。  
これは個人目標の立て方と振り返りのやり方を直したほうがよさそう（完全に見返していなかった）。
別途立てている毎月の個人目標にOSS活動などをしっかり組み込む必要がある。

## Keep
- 単純なコミット数を増やすことができた
- 実用的なOSSを作ることができた
- 自作OSSにいくつかPRをもらえた
- CicleCIの新世代イメージなど、技術キャッチアップもできた
- Terraform Providerを自作するなど、（自分的に）新しい技術を使ってコーディングできた

自作OSSをいくつか作れたのは収穫。もう少し幅を広げていろいろ作っていきたい。

## Problem
- feature PRをOSSに出せていない
- 去年のTryをまったく達成できていなかった
- 去年のProblemとほぼ同じProblemになっている

このProblemは厳しいな…。同じ失敗を繰り返さないようにする。

## Try
- 業務で貢献してprivateコミットを増やす
- OSSに対して機能追加のPRを6個以上出す
- 素振りプロダクトでReactを触ってみる
- 素振りプロダクトでo11yまわりの技術をいろいろ触ってみる

覚悟を決めて個人AWSの稼働を始めたので、来年はその上でいろいろ技術を試していきたい。

# 終わりに
毎年振り返っているが、一度GItHubクエリを覚えれば毎年同じ振り返りができるのでラクだ。  
4年分の振り返りが貯まっているので、見返したり比較するのも楽しくなってきた。  
来年はもっとOSS活動していきたい。


# 関連
- [2017年振り返り(GitHub編)](/2017/12/27/retrospective-on-github/)
- [2018年振り返り(GitHub編)](/2018/12/15/retrospective-on-github/)
- [2019年振り返り(GitHub編)](/2019/12/14/retrospective-on-github-2019/) 
