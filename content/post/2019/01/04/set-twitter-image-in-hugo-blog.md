+++
title = "Hugoのブログ記事にTwitter Card(アイキャッチ画像)を設定する"
date = 2019-01-04T11:36:40+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["hugo", "blog"]
tags = ["hugo", "twitter", "blog"]
twitterImage = "logos/hugo-logo.png"
+++


HugoでTwitter Card（アイキャッチ画像）を設定したときのメモ。

![card validator](/2019/01/04-twitter-image.png)

<!--more-->

なお利用しているHugoのバージョンは以下。

```bash
$ hugo version
Hugo Static Site Generator v0.50/extended darwin/amd64 BuildDate: unknown
```

Themesは以下を利用している。

- Hugo-Octopress
  - https://github.com/parsiya/Hugo-Octopress


# TL;DR
- Hugo-Octopress ThemesでTwitterカードの画像(`twitterImage`)を設定する
  - https://github.com/parsiya/Hugo-Octopress#twitter
- Hugoのもろもろの設定はGoの`text/template`のお作法を知っておく必要がある
  - https://golang.org/pkg/text/template/
- Hugoの環境変数（？）を使ってルートディレクトリ直下の画像を使うようにした。
  - `<meta name="twitter:image:src" content="{{ print $.Site.BaseURL . }}"/>`
- `archetypes/post.md`を書いておけば既定のTwitter Cardのイメージとして設定しておける
  - https://gohugo.io/content-management/archetypes/

# Hugo-Octopress ThemesでTwitterカードの画像を設定する
Twitter CardとはTwitterにURLが貼られたとき、アイキャッチ画像やタイトルが展開されるように設定する機能のことだ。
Hugoは利用しているテーマごとに各設定の有効化方法が異なる。私のHugoはHugo-Octopressというテーマを使っているのでまずHugo-OctopressでどのようにTwitter Cardを有効化するのか確認する。

- https://github.com/parsiya/Hugo-Octopress#twitter

まずはREADMEに書いてあるとおり、config.yamlでTwitter Cardを有効化する。

[budougumi0617/blog/config.toml](https://github.com/budougumi0617/blog/blob/77576171a8f4ef593e092d7b42a977ad452fa3f3/config.toml#L65-L72)
```toml
    # Twitter card config
    # Enable with this.
    twitterCardEnabled = true
    # Don't include the @.
    # twitterCardSite =
    twitterCardDomain = "budougumi0617.github.io"
    # Don't include the @.
    twitterCardAuthor = "budougumi0617"
```

次にTwitter Cardの画像イメージ(`twitterImage`)設定をする。READMEを確認すると、記事ごとに設定しないといけないらしい。

> After Twitter card is enabled, you can add summary images to your posts in front matter with twitterImage:
> twitterImage: 02-fuzzer-crash.png

また、`twitterImage`は「記事と相対的な場所」に置かないといけないらしい。

> Note: Image URL should be relative to the page, otherwise the final URL will not be correct. In short, image URL should be part of the page bundle. In this case, both index.md and 02-fuzzer-crash.png are in the same root directory. If the image is in a subdirectory of page bundle, it can be added like this:
> twitterImage: images/02-fuzzer-crash.png

私の場合、デフォルト用の画像を用意しておいて`twitterImage`を設定したい。なので「ルートディレクトリからの絶対パス」で画像を指定したいのでこの設定を変更する。


# custom_twitter_card.htmlを変更して`twitterImage`の指定方法を上書きする。
- https://gohugo.io/templates/partials/


Hugoではカスタムした設定ファイルを置いておけばThemesの設定ファイルを上書きできる。
Twitter Cardの設定はcustom_twitter_card.htmlで設定できる。Hugo-OctopressでtwitterImageを指定している部分のコードは以下。

- [https://github.com/parsiya/Hugo-Octopress/layouts/partials/custom_twitter_card.html](https://github.com/parsiya/Hugo-Octopress/blob/e62b793c61e75bae591e19af343f15941b955f1d/layouts/partials/custom_twitter_card.html)

```html
...
  {{ with .Params.twitterImage }}
  <!-- Twitter summary card with large image must be at least 280x150px -->
    <meta name="twitter:card" content="summary_large_image"/>
    <meta name="twitter:image:src" content="{{ print $.Permalink . }}"/>
  {{ else }}
    <meta name="twitter:card" content="summary"/>
  {{ end }}
...
```

この中の`{{ print $.Permalink . }}`を変更するので、まずcustom_twitter_card.htmlをコピーして設定上書き用のファイルを作成する。

```bash
$ cp themes/Hugo-Octopress/layouts/partials/custom_twitter_card.html layouts/partials/
```

以下の変数の説明から`Permalink`を確認すると、ページURLのことだとわかる。また、custom_twitter_card.htmlの記法はGoの`text/template`に則っている。

- Variables and Params
  - https://gohugo.io/variables/
- text/template/
  - https://golang.org/pkg/text/template/

以上より`{{ print $.Permalink . }}`はページURLに`twitterImage`に設定されている文字列を連結(`fmt.Sprint`)しているとわかるのでブログのベースURLから文字列を連結するように変更する。`Variables and Params`から確認すると、`.Site.BaseURL`でブログのベースURLが参照できるので、上書き用のcustom_twitter_card.htmlを以下のように変更した。


- [https://github.com/budougumi0617/blog/layouts/partials/custom_twitter_card.html](https://github.com/budougumi0617/blog/blob/master/layouts/partials/custom_twitter_card.html)

```html
...
  {{ with .Params.twitterImage }}
  <!-- Twitter summary card with large image must be at least 280x150px -->
    <meta name="twitter:card" content="summary_large_image"/>
    <meta name="twitter:image:src" content="{{ print $.Site.BaseURL . }}"/>
  {{ else }}
    <meta name="twitter:card" content="summary"/>
  {{ end }}
...
```

あとは記事に`twitterImage`を設定して`static`ディレクトリ以下に`twitterImage`用の画像を配置しておく。

- https://github.com/budougumi0617/blog/blob/master/content/post/2018/12/27/retrospective-2018.md
- https://github.com/budougumi0617/blog/tree/master/static

`hugo server`コマンドでエラーが出ないか動作確認したあと、Web上のデータを更新する。Twitter Cardの確認には以下のサービスを利用することができる。

- Card validator
  - https://cards-dev.twitter.com/validator

上記に`twitterImage`を設定した記事のURLを入力し、きちんと`twitterImage`が展開されることを確認する。

![card validator](/2019/01/04-twitter-image.png)

# 記事作成時にデフォルト設定として`twitterImage`を設定しておく

Hugoはarchetypes/post.mdを作っておけば`hugo new`コマンドで記事を新規作成したときのデフォルトデータを設定しておくことができる。archetypes/post.mdに以下を記述しておけばよい。

```md
+++
...
author = "budougumi0617"
twitterImage = "twittercard.png"
+++
```


# 終わりに
実は以前にもTwitterCardの設定をしようと思っていたのだが挙動がよくわからず失敗していた。
その時点ではそもそもテーマに含まれていたcustom_twitter_card.htmlの記述が間違っていたので失敗していたらしい。久しぶりに再調査したら以下のコミットがされており、今回の変更方法にも気づくことができた。

- https://github.com/parsiya/Hugo-Octopress/commit/3462bf5e01899ec63d02525bb11ee74f286913c2#diff-2637780f50b837ee6eb9c442beb1076b

```diff
   {{ with .Params.twitterImage }}
   <!-- Twitter summary card with large image must be at least 280x150px -->
     <meta name="twitter:card" content="summary_large_image"/>
-    <meta name="twitter:image:src" content="{{ index . 0 | absURL }}"/>
+    <meta name="twitter:image:src" content="{{ print $.Permalink . }}"/>
   {{ else }}
     <meta name="twitter:card" content="summary"/>
   {{ end }}
```

また、Card validatorというのがあるのを初めて知った。しばらくはこれでどうTwitter Cardが見えるのか投稿後確認しておく。

# 参考
- Hugo-Octopress
  - https://github.com/parsiya/Hugo-Octopress
- Partial Templates | Hugo
  - https://gohugo.io/templates/partials/
- Variables and Params | Hugo
  - https://gohugo.io/variables/
- text/template package
  - https://golang.org/pkg/text/template/
- Card validator
  - https://cards-dev.twitter.com/validator
