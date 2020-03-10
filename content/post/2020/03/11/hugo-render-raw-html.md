+++
title= "Hugo v0.60以上を使うと、Markdown中のHTMLタグが「raw HTML omitted」となって消えてしまう"
date= 2020-03-10T09:03:12+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["hugo"]
tags = ["blog"]
keywords = ["Hugo", "blog", "raw HTML omitted", "Goldmark"]
twitterImage = "twittercard.png"
+++

2020/03/07時点で最新のHugo v0.66.0で`hugo`コマンドを使ってMarkdownファイルからHTMLを生成した。  
すると、Markdown中に含まれていたHTMLタグがすべて`<!-- raw HTML omitted -->`と出力され消えるようになってしまった。  
結論から言うと、Hugo v0.60.0からは設定で明示的に「HTMLタグをそのまま出力する」オプションを設定する必要があった。

<!--more-->

# TL;DR
- Hugoを最新版のv0.66.0にしたらMarkdown文書中のHTMLタグが出力されなくなってしまった
    - 生成後のHTMLファイルをみると、`<!-- raw HTML omitted -->`になっていた。
- Hugo v0.60.0から利用されている`Goldmark`というMarkdown Parserのデフォルト設定はHTMLタグを出力しない
- Markdown中のHTMLタグを出力するには、`config.toml`などで`unsafe`オプションを`true`にする必要がある

```toml
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
```

記事で利用しているHugoのバージョンは次の通り。

```bash
$ hugo version
Hugo Static Site Generator v0.66.0/extended darwin/amd64 BuildDate: unknown
```

# Markdown中のHTMLタグが出力されなくなった。
このブログはHugoという静的サイトジェネレータを使って構築されている。  
HugoはMarkdownでWebページを記述することができる。  
私のブログ記事ではAmazonへのリンクや次のようなリッチなリンクを生成するため、Markdown記事内に`<div>`タグや`<iframe>`タグが書いていた。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://gohugo.io/" data-iframely-url="//cdn.iframe.ly/Btwbn5A?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

（ちなみにこのようなリンクはiframelyというサービスで生成している。）

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://iframely.com/" data-iframely-url="//cdn.iframe.ly/KGpjY3"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


すこし気になるところがあったので、`brew upgrade hugo`コマンドで`hugo`コマンドを2020/03/07時点最新のv0.66.0にした。  
その結果、ブログ記事となるMarkdown内にそのまま書いていたHTMLタグがサイトに公開するHTMLから消えてしまった。  

生成されたHTMLを見ると、`<div>`タグや`<iframe>`タグが書いていたところがすべて以下のようになっていた。


```diff
<p>そして、<code>claat</code>はGoogleが提供している公式ツールだ。<code>claat</code>を使うとGoogle DocからこのCodelabs形式のチュートリアル・ハンズオンを簡単に作成することができる。</p>
-
- <p><div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/googlecodelabs/tools" data-iframely-url="//cdn.-iframe.ly/fSusqIF"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script></p>
-
+ <!-- raw HTML omitted -->
```

# Hugo v0.60.0からMarkdown中のHTMLタグがオミットされるようになった
調べてみたところ、Hugo自体の設定が変わったらしい。  
Hugo v0.60.0から新しく採用された`Goldmark`というMarkdownパーサーはデフォルト設定でMarkdonw中のHTMLタグを無視する。

- Configure Markup | Hugo
    - https://gohugo.io/getting-started/configuration-markup/#goldmark

> Goldmark is from Hugo 0.60 the default library used for Markdown.

> unsafe
> By default, Goldmark does not render raw HTMLs and potentially dangerous links. If you have lots of inline HTML and/or JavaScript, you may need to turn this on.

なので、この`unsafe`オプションを`true`にすることで、再びMarkdown内のHTMLタグがHugoで生成された記事内でも表示されるようになった。  
私の場合は`config.toml`ファイルに設定を書いていたので、以下のような設定を追加した。


```toml
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
```

# 終わりに
新しい機能を追加するとき、デフォルト設定は挙動が変わらない方向にしておかないと、アップデートしたユーザーが困ったり、びっくりしたりするという教訓を得た。

# 参考
- [Hugo](https://gohugo.io/)
- [iframely](https://iframely.com/)
- [Configure Markup | Hugo](https://gohugo.io/getting-started/configuration-markup/#goldmark)
