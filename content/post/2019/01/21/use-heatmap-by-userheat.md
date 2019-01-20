+++
title= "HugoブログにUser Heatを設定してヒートマップ分析をする"
date= 2019-01-21T00:21:39+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["hugo", "blog"]
tags = ["hugo", "analytics", "SEO"]
keywords = ["Hugo", "Blog", "Heat Map", "userheat"]
twitterImage = "2019/01/heatmap-title.png"
+++

User Heatという無料サービスを使ってブログのヒートマップを導入してみた。
ヒートマップ分析を使うことで記事のどこに注目が集まっているかなどを分析することができる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="http://userheat.com" data-iframely-url="//cdn.iframe.ly/api/iframe?url=https%3A%2F%2Fuserheat.com&amp;key=bc7b75ad5adf2dc500d4b0dee9f92949"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
<!--more-->


# TL;DR
- User Heatは月間30万PVまで無料で使える解析ツール
  - https://userheat.com
- 訪問者がページのどこの段落を熟読したか、どこをクリックしたかをビジュアル分析できる
- Hugoで利用する場合は`layouts/partials/header.html`にHTMLタグを埋め込むだけ


# ヒートマップ分析とは
- https://userheat.com/whatheatmap

ヒートマップ分析とは該当サイトを閲覧するユーザーの行動を分析する。「どのページが見られたか？どこから来たか？」だけでなく以下の行動までを分析する。

- ページのどこまでスクロールされたか？
- どこをクリックしたか？
- どの部分が読まれていたのか？

これらの情報を可視化することで、サイト内のページのどの部分まで読まれているのか、ページ中のどの文章が注目されているのかを可視化することが出来る。

# 無料ヒートマップ解析ツール User Heatを使って分析してみる
- https://userheat.com

User Heatは[カックさん](https://twitter.com/kakakakakku)から教えていただいた上記のヒートマップ分析を行えるサービスだ。
月間30万PVまでのサイトならば無料で利用することができる（後述する制限がある）。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="http://userheat.com" data-iframely-url="//cdn.iframe.ly/api/iframe?url=https%3A%2F%2Fuserheat.com&amp;key=bc7b75ad5adf2dc500d4b0dee9f92949"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# Hugoで利用する場合は`layouts/partials/header.html`にHTMLタグを埋め込むだけ
- https://gohugo.io/templates/partials/

Hugoで利用する場合はカスタムヘッダーのHTMLを用意して発行したHTMLタグを貼り付けておくだけでよい。
なお、はてなブログやWordPressなどへの導入方法はUser Heatの公式ページに解説されている。


2019年1月現在私のブログで利用している[Hugo-Octopress](https://themes.gohugo.io/hugo-octopress/)テーマでは以下のように設置した。

- Add custom header(set userheat)
  - https://github.com/budougumi0617/blog/commit/bdbc7731d3be30e7d647bd5c0fb78234d40b88c4

- User Heatの[解析タグ発行](https://userheat.com/groups)ページからHTMLタグをコピーしておく
- `$(REPOSITORY_ROOT)/layouts/partials/header.html`に既存のheader.htmlをコピペしておく。
  - `cp themes/Hugo-Octopress/layouts/partials/header.html layouts/partials`
- `layouts/partials/header.html`にUser Heatから取得した解析タグを貼り付けておく

```diff
    {{ with .Site.Params.twitterCardEnabled }}
      {{ partial "custom_twitter_card.html" $ }}
    {{ end }}

+   <!-- User Heat Tag -->
+   <script type="text/javascript">
+     (function(add, cla){window['UserHeatTag']=cla;window[cla]=window[cla]||function(){(window[cla].q=window[cla].q||[]).push(arguments)},window[cla].l=1*new Date();var ul=document.createElement('script');var tag = document.getElementsByTagName('script')[0];ul.async=1;ul.src=add;tag.parentNode.insertBefore(ul,tag);})('//uh.nakanohito.jp/uhj2/uh.js', '_uhtracker');_uhtracker({id:'HOGEHOGE'});
+   </script>
+   <!-- End User Heat Tag -->
  </head>
```

あとはブログを再生成して更新する。Chromeでブログを開き、Dev ToolsでHeaderを確認しながらスーパーリロードを繰り返し、上記のスクリプトが確認できれば設置が完了している。


User HeatのWebページでも確認できる。設置が完了していれば解析が始まっている旨が表示される。注意書きの通り設置後200, 300PVが貯まるまで待つ。
サイト全体のPV数ではなく、各ページごとのPV数なので私のようなサイトだと１週間くらいかかった気がする。

- 解析結果の一覧
  - https://userheat.com/summaries

![データの集計中](/2019/01/21-start-analytics.png)


# 分析を見て
設置してしばらく経ち、記事によってはデータも蓄積されたので早速内容を確認してみた。

![ページ一覧](/2019/01/21-userheat-summary.png)

終了エリアを確認すると、ユーザーがどの部分まで読んで離脱したのかわかる（「このエリアで何%のユーザーが離脱したのか」がわかる離脱エリアのビューもある）。
次のキャプチャは50%のユーザーがここまで読んだ、ということがわかる。

![終了エリア](/2019/01/21-escape.png)


熟読エリアをみると、ページのどの部分に注目が集まっているのかわかる。意図通りの場所に注目が集まっていない場合、なんらかのリライトが必要になるかもしれない。
私の記事の場合、TL;DRに注目があつまっていたり、コードの解説に対して注目が集まっていたのでそれなりに意図通りの記事が書けていそうだった。

![熟読エリア](/2019/01/21-heatmap.png)

# 無料利用だと制限もある
UserHeatは無料サービスなので、解析出来るページの長さに制限がある。
私のブログの場合、キャプチャを添付した記事などは長さ制限に引っかかっていたようで、分析が記事の途中で切れてしまっていた。
本格的に分析を使いたい場合は、有料のUserInsightを利用する必要がある。

- ヒートマップが途中できれてしまう
  - https://userheat.com/stage/help/#breakup


# Google AnalyticsユーザーならばPage Analyticsで簡易分析も出来る。
ここまでリッチな機能はないが、Google Analyticsを利用しているならばChrome拡張機能のPage Analyticsを利用することもできる。


<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://chrome.google.com/webstore/detail/page-analytics-by-google/fnbdnhhicmebfgdgglcdacdapkcihcoh" data-iframely-url="//cdn.iframe.ly/api/iframe?url=https%3A%2F%2Fchrome.google.com%2Fwebstore%2Fdetail%2Fpage-analytics-by-google%2Ffnbdnhhicmebfgdgglcdacdapkcihcoh&amp;key=bc7b75ad5adf2dc500d4b0dee9f92949"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

利用は簡単で拡張機能インストール後、Chromeに紐付いているGoogle Analyitcsで解析しているサイトを開けば自動的にサイト内リンクが踏まれた回数などを表示してくれる。

![Page Analytics](/2019/01/21-page-analytics.png)

# 終わりに
2018年は「ブログは自分用のメモ」とある意味妥協して「情報としてのの質」を深く考慮していなかった。
2019年は第三者にとっても読みやすく有益な情報となるブログ記事を書いていきたい。


# 参考
- User Heat
  - https://userheat.com
- Page Analytics (by Google)
  - https://chrome.google.com/webstore/detail/page-analytics-by-google/fnbdnhhicmebfgdgglcdacdapkcihcoh



