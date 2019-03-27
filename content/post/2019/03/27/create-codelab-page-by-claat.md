+++
title= "claatを使ってCodelabs形式のオリジナルハンズオンを作る"
date= 2019-03-27T17:24:43+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["google"]
tags = ["codelab", "claat"]
keywords = ["claat", "Codelab", "Google", "資料作成"]
twitterImage = "2019/03/27_codelab.png"
+++

先日golang.tokyoのコンテンツとしてハンズオン資料を作成した。  
claatというツールを使うと、Google Docに書くだけでGoogle Codelabsと同じ体裁のコンテンツが作成できるので紹介する。

- https://budougumi0617.github.io/claat-sample

![My Codelab](/2019/03/27_codelab.png)

<!--more-->


# TL;DR
- Google Docと`claat`だけでGoogleと同じフォーマットのハンズオンページを作成することができる。
- 以下のフォーマットに則ってGoogle Docを書くだけ
  - [Codelab Formatting Guide](https://g.co/codelabs/guide)
- [Preview Codelab Chrome extension](https://chrome.google.com/webstore/detail/preview-codelab/lhojjnijnkiglhkggagbapfonpdlinji)を使えばプレビューを見ながら作業できる
- `claat`コマンドを使えばGoogle Docから1枚のHTMLファイルを生成できる
  - https://github.com/googlecodelabs/tools/tree/master/claat
  - Google DocのIDを渡すだけでHTML(1ファイル)を生成してくれる
       - `claat export $DOC_ID`
- 生成したHTMLをGitHub Pagesなどに公開するだけでハンズオンが公開できる


Codelab形式のハンズオンを作成するためのサンプルのGoogle Docは以下。

- https://docs.google.com/document/d/17d7hzKZVbiAvEQ882ozn19DI6Sy-DWbXLJnv_EgK5xY/edit?usp=sharing

上記のGoogle Docから生成したHTMLは以下になる。

- https://budougumi0617.github.io/claat-sample

# Google Codelabsとclaat
CodelabsはGoogleが提供しているチュートリアルやトレーニング資料の配布サイトだ。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://codelabs.developers.google.com/" data-iframely-url="//cdn.iframe.ly/irV0q8"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

GCPやGo、Androidなど膨大な量のチュートリアルが用意されているので、なにかしら一度はやったことがあるのではないだろうか(下図はGoogleが提供しているKubernetesのトレーニング)。  

![Real Codelab](/2019/03/27_real_codelab.png)

そして、`claat`はGoogleが提供している公式ツールだ。`claat`を使うとGoogle DocからこのCodelabs形式のチュートリアル・ハンズオンを簡単に作成することができる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/googlecodelabs/tools" data-iframely-url="//cdn.iframe.ly/fSusqIF"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Google Docではなく、Markdownでも作成できるらしいのだが、Google Docで作るほうが楽なので今回はDocを使ってCodelab形式の資料を作る方法を紹介する。

# フォーマットに従ってGoogle Docを作成する
まずは元となるGoogle Docを作成する。と言っても、既定のルールにしたがって文書を装飾していくだけでよい。  
例を挙げると以下のようなルールだ。

- 新しいセクションを始めたいときはセクションタイトルに「見出し1」を使う
- セクションの所要時間は`Duration: 9:99`というように時間を書いて、「dark gray 1」色にしておく

フォーマットを使ったサンプルのDocは以下になる。

- https://docs.google.com/document/d/17d7hzKZVbiAvEQ882ozn19DI6Sy-DWbXLJnv_EgK5xY/edit?usp=sharing

![Google Doc](/2019/03/27_google_doc.png)

記載した装飾以外にもダウンロードボタンの設置や注釈を書くなど、様々な装飾を行なうことができる。
すべての装飾ルールは以下のガイドに記載されている。

- Codelab Formatting Guide
  - https://github.com/googlecodelabs/tools/blob/master/FORMAT-GUIDE.md

基本的にはこのMarkdownのガイドを見ながら作ればよい。ただ、同じ内容が記載されているGooge Docを見ながらのほうが楽だろう。

- Codelab Formatting Guide
  - https://g.co/codelabs/guide

文字的には同じ内容が書いてあるが、GoogleDocの方は形式が設定された編集項目をコピペしてそのまま使える。なのでGoogle Docのフォーマットを見ながら作ったほうが圧倒的に速い。
アクセス権限が必要なGoogle Docだが、リクエストをすればすぐ閲覧許可がもらえる。

## ChromeでプレビューしながらDocを執筆する。
Google Docで文書を作成している間は、Chrome拡張機能を使うと編集中のGoogle Docから出力されるCodelabの構成をすぐ確認できる。

- Preview Codelab
  - https://chrome.google.com/webstore/detail/preview-codelab/lhojjnijnkiglhkggagbapfonpdlinji

Google Docの準備が終わったらあとはHTMLを生成するだけだ。

# `claat`の使い方
HTMLの生成には前述の`claat`コマンドを利用する。`claat`は以下のページからDLできる（`go get`してもよい）。

- https://github.com/googlecodelabs/tools/tree/master/claat

`claat`コマンド自体の使い方は簡単だ。
DL後、`claat export [Google DocのID]`と実行する。引数は該当するGoogle DocのID部分を記載するだけだ。

 ```bash
# https://docs.google.com/document/d/17d7hz...EgK5xY というURLのGoogle Docの場合
$ claat export 17d7hz...EgK5xY
Authorize me at following URL, please:

https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=1...


Code:
```

初回実行時は上記のように認証が求められる。コマンドラインに出力されるURLでログイン後、発行されるコードを入力すれば良い。
生成に成功するとカレントディレクトリ配下にHTMLとJSONファイルが(画像を含む場合はimgディレクトリも)生成される。

```
$ tree claat-sample
claat-sample
├── codelab.json
├── img
│   └── 12c740002e6e3d6.png
└── index.html
```

生成が終わったあとは、index.htmlがあるディレクトリに移動し、`claat serve`コマンドを実行すればブラウザで結果を確認することができる。  
内容に問題なければGitHub PagesやS3などに出力ファイルを配置するだけでオリジナルのCodelabを公開することができる。

- https://budougumi0617.github.io/claat-sample

![My Codelab](/2019/03/27_codelab.png)

# 終わりに
golang.tokyoとしてハンズオンを作成したときに`claat`コマンドの使い方の情報があまりなかったのでまとめた。  
ハンズオンやトレーニング資料は学習内容が大事だとは思うがやはり体裁も大事だ。  
Googleと同じ体裁で資料を簡単に作れるというのは、体裁を気にせず教材内容に時間を当てられるので非常にありがたい。

# 参考
- Google Codelabs の執筆者用ツールの使い方
  - https://daisuzu.hatenablog.com/entry/2018/07/11/024204
- Google Codelabs
  - https://codelabs.developers.google.com/
- claat
  - https://github.com/googlecodelabs/tools/tree/master/claat
- Codelab Formatting Guide
  - https://g.co/codelabs/guide
