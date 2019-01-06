+++
title= "HTTPSで公開しているHugoで一部のページがHTTPでページ遷移してしまう #hugo"
date= 2019-01-06T17:14:48+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["hugo", "blog"]
tags = ["hugo", "blog"]
keywords = ["hugo"]
twitterImage = "logos/hugo-logo.png"
+++

Hugoで公開しているこのブログのサイドバーのリンクの一部をクリックすると、HTTPSで公開しているはずがHTTP通信となって遷移していた。  
HTTPS通信でページ遷移するように直したときのメモ。

<!--more-->

# TL;DR
- Chrome DevToolsで通信内容を確認する
- リンク先アドレスの指定が微妙に間違っていたので、リダイレクトが発生していた
  - リダイレクト時に、HTTPでページ遷移を行なっていた
- 正しいパスに修正するとHTTPSのままページ遷移するようになった
- 原因はわからなかった

# Chrome Developer Toolで通信内容を確認する
私のこのブログは2019/01/06現在サイドバーにTagsページとCategoryページへのリンクを貼っている。
この２つのページへのリンクがマウスホバーをしたときには`https://budougumi0617.github.io/tags`とHTTPSのリンクになっているのだが、実際にリンクをクリックすると、`http://budougumi0617.github.io/tags/`とHTTPで画面遷移してしまっていた（この時点で分かる人は分かったかもしれない）。

![マウスホバー時の情報](/2019/01/06-hover-information.png)


ひとまずページ遷移中に何が起きているか確認するため、Chrome DeveToolsでNetworkタブで通信内容を確認してみた。リクエストとリソースの流れを確認した結果、途中でリダイレクトが発生していた。以下は`https://budougumi0617.github.io/tags`へのアクセス時のHeadersの情報の一部だ。

![DevToolsの情報](/2019/01/06-devtools.png)

```
General
    Request URL: https://budougumi0617.github.io/tags
    Request Method: GET
    Status Code: 301
    ...

Response Headers
    accept-ranges: bytes
    access-control-allow-origin: *
    ...
    location: http://budougumi0617.github.io/tags/
    server: GitHub.com
    status: 301
    ...
```

Hugoで生成したタグページの正式なURLは`${BASE_URL}/tags/index.html`となる。前述のリンク先に指定していたURLは`${BASE_URL}/tags`だったため、`/`が付与された`${BASE_URL}/tags/`(`index.html`)へのリダイレクトが発生していた。リダイレクトなしでページ遷移すれば良さそうだ。

# サイドバーのリンクの設定を修正する
サイドバーの設定はTOMLファイルで以下のようになっている。

- [config.toml](https://github.com/budougumi0617/blog/blob/0e74c26179d16d343c8f98f9dd65a3b7102ea6ca/config.toml#L118-L126)

```toml
[[menu.sidebar]]
  Name = "Tags"
  URL = "/tags"
  weight = 1

[[menu.sidebar]]
  Name = "Categories"
  URL = "/categories"
  weight = 1
```

ここのURLの指定に明示的に`/`をつけ、`URL = "/tags/"`としたところ、リダイレクトが発生しなくなり、HTTPS通信でページ遷移できるようになった。

# 終わりに
なぜリダイレクト時にHTTPS通信がHTTP通信になってしまうのか？調査すればOSSへのPRチャンスかとも思っていたのだが、すぐに原因はわからないかった。悔しいが目的の修正はできたので今回はここで終わりにしておく。  
まったく本編と関係ないのだが、今まで「Chrome Developer Tools」だと思っていたが、正式名称は「Chrome DevTools」のようだ。

https://developers.google.com/web/tools/chrome-devtools/

# 関連
- [Hugoのブログ記事にTwitter Card(アイキャッチ画像)を設定する #hugo](2019/01/04/set-twitter-image-in-hugo-blog/)
- [HugoでOGP(Facebook用のアイキャッチ画像)を設定する #hugo](/2019/01/05/set-ogp-in-hugo-blog/)
