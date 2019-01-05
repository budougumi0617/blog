+++
title= "HugoでOGP(Facebook用のアイキャッチ画像)を設定する #hugo"
date= 2019-01-05T09:42:44+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["hugo", "blog"]
tags = ["blog", "ogp", "facebook"]
keywords = ["Hugo", "SEO", "OGP", "Facebook"]
twitterImage = "logos/ogp.png"
+++

Hugoで各記事にThe Open Graph protocol(OGP)を設定したときのメモ。

- The Open Graph protocol
  - http://ogp.me/

![ogp](/2019/01/05-ogp.png)

<!--more-->


# TL;DR
- Facebookでシェアされたときのアイキャッチ画像などはThe Open Graph protocolで設定できる
- ヘッダーに`<meta property="og:xxx"...>`と設定すればよい
- Twitter Cardの情報と同じなので、Hugoの場合はcustom_twitter_card.htmlを変更しておけばよい
- Facebookに開発者登録をしておけばシェアデバッガーで設定状態を確認することが出来る
  - https://developers.facebook.com/tools/debug/sharing/

# Facebookでシェアされた場合に必要なOZGPを設定する
先日以下の記事でTwitterで投稿されたときにアイキャッチ画像が表示されるようにTwitter Cardを設定した。

- [Hugoのブログ記事にTwitter Card(アイキャッチ画像)を設定する](/2019/01/04/set-twitter-image-in-hugo-blog/)

今回はFacebookに投稿されたときにアイキャッチ画像が表示されるように、The Open Graph protocol(OGP)の設定をする。OGPの仕様の詳細は以下のサイトで確認できる。

- The Open Graph protocol
  - http://ogp.me/

# Hugoのcustom_twitter_card.htmlを修正する
Hugoの場合、対応方法はTwitter Cardに対応したときとほぼ同じだ。layouts/partials/custom_twitter_card.htmlを用意して、ヘッダーにメタ情報を埋め込む。
custom_twitter_card.htmlに埋め込むのはTwitter Cardに埋め込む情報とほとんど同じデータを利用するためだ。

- Partial Templates
  - https://gohugo.io/templates/partials/

`+`をつけた行がOGPの対応で変更した内容だ。ほとんどTwitter Cardの情報をメタ属性を変更して流用しているだけだ。気をつける点としては`name`属性ではなく、`property`属性で指定しないと認識されない。

- [layouts/partials/custom_twitter_card.htmlの編集結果](https://github.com/budougumi0617/blog/compare/a222edb...d5136027de10b66419de4add943f6808386eadde)

```html
  <!-- Modified from _internal/twitter_card.html -->
  {{ if .IsPage }}
    {{ with .Params.twitterImage }}
+   <!-- The Open Graph protocol data -->
+     <meta property="og:image" content="{{ print $.Site.BaseURL . }}"/>
    <!-- Twitter summary card with large image must be at least 280x150px -->
      <meta name="twitter:card" content="summary_large_image"/>
      <meta name="twitter:image:src" content="{{ print $.Site.BaseURL . }}"/>
    {{ else }}
      <meta name="twitter:card" content="summary"/>
    {{ end }}
+   <!-- The Open Graph protocol data -->
+   <meta property="og:url" content="{{ $.Permalink }}"/>
+   <meta property="og:type" content="article"/>
+   <meta property="og:title" content="{{ .Title }}"/>
+   <meta property="og:description" content="{{ with .Description }}{{ . }}{{ else }}{{ if .IsPage }}{{ .Summary }}{{ else }}{{ with .Site.Params.description }}{{ . }}{{ end }}{{ end }}{{ end }}"/>
    <!-- Twitter Card data -->
    <meta name="twitter:title" content="{{ .Title }}"/>
    <meta name="twitter:description" content="{{ with .Description }}{{ . }}{{ else }}{{ if .IsPage }}{{ .Summary }}{{ else }}{{ with .Site.Params.description }}{{ . }}{{ end }}{{ end }}{{ end }}"/>
      {{ with .Site.Params.twitterCardSite }}<meta name="twitter:site" content="@{{ . }}"/>{{ end }}
    {{ with .Site.Params.twitterCardDomain }}<meta name="twitter:domain" content="{{ . }}"/>{{ end }}
    {{ with .Site.Params.twitterCardAuthor }}<meta name="twitter:creator" content="@{{ . }}"/>{{ end }}
  {{ end }}
```

# 対応後はシェアデバッガーで確認する
上記の対応が正しくできているかは、シェアデバッガーを使って確認出来る。シェアデバッガーは以下のURLから利用できる。調べたいページのリンクを入力してデバッグボタンをクリックするだけだ。

- シェアデバッガー
  - https://developers.facebook.com/tools/debug/sharing/

シェアデバッガーの利用にはFacebookへの開発者登録が必要なので、以下のページから行う(私は遠いはるか昔に登録していたので今回登録フローはせずに利用できた)。

- Facebook for Developers
  - https://developers.facebook.com/

Web上のデータを更新してシェアデバッガーで確認したところ、正しく対応できたことが確認できた。

![ogp](/2019/01/05-ogp.png)

# 終わりに
添付したスクショをよく見てもらうとわかるが、実は「次のプロパティは必須です: fb:app_id」というシェアデバッガーのエラーは消えていない。
調査してみると、fb:app_idがついているとFBのAPIを利用できるようになる == FBの解析機能(どれくらい"いいね"されたとか)を使えるようになるらしい。が、今回は個人のブログでそこまでマーケティングなどは気にしていないので対応を保留した。

- アプリ開発
  - https://developers.facebook.com/docs/apps#register

# 参考
- The Open Graph protocol
  - http://ogp.me/
- シェアデバッガー
  - https://developers.facebook.com/tools/debug/sharing/
- アプリ開発
  - https://developers.facebook.com/docs/apps#register

# 関連
- [Hugoのブログ記事にTwitter Card(アイキャッチ画像)を設定する](/2019/01/04/set-twitter-image-in-hugo-blog/)
