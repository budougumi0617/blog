+++
title= "PixelaのGo用APIクライアントライブラリを作り始めた #pixela"
date= 2021-01-03T08:00:27+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["pixela","actions","golang"]
keywords = ["pixela","github actions"]
twitterImage = "twittercard.png"
+++

GoのPixela APIクライアントライブラリを作り始めた。  
最低限自分が今必要なAPIはカバーできたのでpublicに公開した。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/pixela" data-iframely-url="//cdn.iframe.ly/lYXbTqm?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- PixelaのGo用APIクライアントライブラリを作りはじめた
    - https://github.com/budougumi0617/pixela
    - https://pixe.la/
    - https://docs.pixe.la/
- ひとまずテストは最小限のAcceptance Testを書いている
    - 実APIを叩いているのでモックの更新漏れなどもない
- 自動でリリースタグを作るようにしているのでリリースがとてもラク
    - GoReleaserは使っていない
- リズムよく開発できるので個人開発では（でも）最初にテストとCI/CDを書いておくのがオススメ


# Pixelaについて
Pixelaは[@a-know](https://twitter.com/a_know)さんが提供しているWebサービスだ。  
日々のさまざまな活動量をGitHubのような鮮やかなグラフにすることができる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://pixe.la" data-iframely-url="//cdn.iframe.ly/q3gtB1F?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

詳細なAPIドキュメントも公開されており、エンジニアならcurlなどでも簡単に利用することができる。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://docs.pixe.la/" data-iframely-url="//cdn.iframe.ly/8xwV0qR?media=0"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# なぜ作るのか？
昨年、PixelaのグラフをTerraformから操作するためのTerraform Providerを作った。

[TerraformでPixelaのグラフを宣言できるProviderを作った #pixela](/2020/12/11/terraform_provider_pixela/)

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/terraform-provider-pixela" data-iframely-url="//cdn.iframe.ly/zjuslmC?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Terraform ProviderはGoで実装するのだが、この中でPixelaのAPIを叩くために3rdパーティのAPIクライアントを使っていた。  
PixelaのGo APIクライアントはいくつかあるのだが、いろいろ思うところがあったので自作することにした。

# budougumi0617/pixela
今回作ったPixela API実行用のAPIクライアントは次のとおり。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/terraform-provider-pixela" data-iframely-url="//cdn.iframe.ly/zjuslmC?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

まだグラフを操作するメソッドしかないが、主に次の点を満たしたくて自作した。

- `context.Context`対応
- 素直なメソッド呼び出し
- v1.21.0で増えた新しいエンドポイントをサポートしている
    - https://github.com/a-know/Pixela/releases/tag/v1.21.0
    - https://docs.pixe.la/entry/get-a-graph-def


また、APIクライアントライブラリの作成に合わせてTerraform Providerも更新した。
https://github.com/budougumi0617/terraform-provider-pixela/releases/tag/v0.0.6

# Acceptance Testという安心
今回はズルして単体テストはほとんど書いていない。  
実際のAPIを叩いてテストしているので「モックが古い仕様のままだった」みたいな不安はない。

- https://github.com/budougumi0617/pixela/blob/master/e2e_test.go

バリデーションなど、細かい実装を一切していないので、そういった実装書き始めるときは単体テストでよさそう。  
Terraform Providerの方もAPIを直接叩くAcceptance Testを書いてあったのでデグレの心配なく更新できた。  


# 自動でリリースタグを作るようにしているのでリリースがとてもラク
release-it npmを使って自動リリースする設定をしている。
次の2ファイルを置いておくだけでmasterブランチにpushすれば自動で新しいバージョンが切られる。

- https://github.com/budougumi0617/pixela/blob/master/.release-it.json
- https://github.com/budougumi0617/pixela/blob/master/.github/workflows/release.yml

上記ファイルについての説明は以前所属会社のブログに記載した。
<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fautomatic-release-on-github-actions" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

APIクライアントライブラリ（バイナリ配布する予定がない）なのでGoReleaserは使っていない。  
いちいち「いまのバージョンなんだっけ？」「バージョンタグはv付けてたっけ？」なんて考えなくてよいので非常にラクだ。

# 個人開発でも最初にテストとCI/CDを書いておくのがオススメ
昨年から趣味でコードを書く時間が深夜になったのでどうしても眠気との戦いになっている。  
ウトウトしながらコーディングしていてもテストがちゃんとあればエンバグすることは少ない。  
また、レビューしてもらえるわけでもないのでCIでテストと静的解析を流しておけば独りでも最低限のクオリティを確認しながら開発ができる。

他方、個人的に個人開発しているときは「機能実装が終わった（キレイに実装できた）」というタイミングがピークだ。  
なので、そのあとに「バージョン番号を思い出してインクリメントしたtagを切って…」なんてやる時間は面白くない。
Terraform Providerのリリースになると[バイナリをzipにしたあと証明書で署名する][tp_release]必要もあり、絶対に手作業したくない。  
だから最初に自動リリースやデプロイを整備しておくとモチベーションの高い状態をキープしたまま実装を続けられるのでよい。

[tp_release]: https://www.terraform.io/docs/registry/providers/publishing.html#manually-preparing-a-release

# 終わりに
- https://github.com/budougumi0617/pixela

PixelaのAPIクライアントライブラリを作った話をまとめた。  
別のツールを作る必要があるので一度開発はストップしてしまうのだが不足しているAPIも少しずつカバーしていきたい。  
最初なのでDRYはまったく考えずに実装したので、それなりに実装が進んだらリファクタリングも始めたい。

# 参考
- https://github.com/budougumi0617/pixela
- https://pixe.la/ja
- https://docs.pixe.la/
- https://github.com/budougumi0617/terraform-provider-pixela
- [TerraformでPixelaのグラフを宣言できるProviderを作った #pixela](/2020/12/11/terraform_provider_pixela/)
