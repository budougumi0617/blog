+++
title= "[書評] ghq handbookを読んでghqコマンドに再入門する"
date= 2020-01-11T21:31:17+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["review"]
tags = ["ghq","book"]
keywords = ["ghq","書評","ghq handbook"]
twitterImage = "2020/01/11_ghq-handbook.png"
+++

ローカルでソースコードを効率よく管理するためのツールとして、`ghq`コマンドがある。  
その`ghq`コマンドの使い方をメンテナの[@songmu](https://twitter.com/songmu)さんが執筆したghq handbookを読んだので感想をまとめる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="http://leanpub.com/ghq-handbook" data-iframely-url="//cdn.iframe.ly/hPd1pAJ"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


<!--more-->

# 所感
コンパクトで大変わかりやすい一冊だった。私は少なくても3年以上`ghq`を使っている。
すでに`ghq`コマンドを利用している人も、初めて使う人もぜひ読んでおくべき1冊だ。手を動かさなければボリューム的には30分もかからず読み終わるだろう。  
やりたいことベースで節が書かれているので、さっと気になるところから読めるのもよかった。  
「どんなことをしたいのか？（例：会社のリポジトリは別の場所でディレクトリで管理したい etc）」を思い浮かべながら目次をたどれば読むべきページががわかるだろう。  
私はあまり英語が得意でなかったり、`zshrc`をうまく書けないので、`peco`や`fzf`のような`fuzzy finder`との連携のベストな設定などを知れてよかった。

# どんな本なのか
`ghq`コマンドというのは、GitHubなどのVCSサイトからのリポジトリの取得と、ローカルPC内のリポジトリ管理を劇的にラクにしてくれるツールだ。  
私は、Go以外のリポジトリも全て`ghq`コマンドで管理している。いちいちディレクトリ構造を頭に浮かべながら`cd`コマンドで移動することはなくなった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/x-motemen/ghq" data-iframely-url="//cdn.iframe.ly/MssyxU4?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`ghq`コマンドのv1.0.0バージョンリリースに合わせて、メンテナの[@songmu](https://twitter.com/songmu)さんが使い方をまとめてくださったのが本書だ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://songmu.jp/riji/entry/2020-01-05-ghq-v1.html" data-iframely-url="//cdn.iframe.ly/aUbn23p?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

ghq handbookは`leanpub.com`というサイトから購入することもできるし、GitHub上でMarkdownを読むこともできる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="http://leanpub.com/ghq-handbook" data-iframely-url="//cdn.iframe.ly/hPd1pAJ"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# なぜ読もうと思ったのか
私は今までなんとなく`ghq get`コマンドと`ghq list + peco`連携を使っていたので、改めてちゃんとした使い方を知るために読んだ。  
私にとって`ghq`コマンドは、`jq`や`rg`、`peco`/`fzf`コマンドと並んで無くてはならないコマンドラインツールだったので日々の感謝の気持ちを込めて`leanpub.com`で購入させてもらった。

余談だが、初めてleanpubを使ってみたが`send to kindle`（iPadなどのKindleアプリに電子書籍を送信する）がサイト上からワンクリックで簡単に出来たり、なかなか良かった。  
（Amazon上で予めleanpubからのメール受信を許可しておくのと、メルアドを登録したから少し時間が必要）

# 読んでわかったこと
さっそく本を購入するにあたって最新の`ghq`コマンドにした。
macOSならば`brew upgrade ghq`コマンドで簡単にバージョンアップが可能だ。  

## 複数root設定時のzhqコマンドとpecoコマンドの連携方法
まず、`zsh`でいい感じの`peco`コマンドとの連携設定を知ることが出来たのがよかった。  
最近仕事用と私用のディレクトリ（`ghq root`）を分けて`ghq`コマンドを使おうと思っていたのだが、うまく設定できていなかった。
本書で無事解決することが出来た。  
さっそく自分のdotfilesの`ghq`関連の設定を更新した。

```bash
function peco-src () {
    local repo=$(ghq list | peco --query "$LBUFFER" --initial-filter=Fuzzy)
    if [ -n "$repo" ]; then
        repo=$(ghq list --full-path --exact $repo)
        BUFFER="cd ${repo}"
        zle accept-line
    fi
    zle clear-screen
}
zle -N peco-src
bindkey '^]' peco-src
```

## ghq look（ghq get --look）コマンドの誤った使い方
本を読むまで、「`look`コマンドはシェルを改めて起動するから嫌な感じだなあ」と思っていたのだが、単にユースケースを間違えていただけだった（今まで`ghq look`コマンドを使った間違ったディレクトリ移動方法を使っていただけだった）。なお、v1バージョンの`ghq`コマンドでは`ghq look`コマンドは廃止されている。  
v1では`ghq get --look`コマンドのみになっているので、`ghq look`コマンドを使った誤った移動を`rc`ファイルに記載している人は動かなくなっているだろう。

その他でも、自分のリポジトリを取得するとき（会社のPCで取得するなど、意外と頻度がある）、`ownername`（username）を省略できるのは知らなかった。
また、レシピ集としてワンライナーや他のツールと組合わせた例も提案されていた。これをヒントに自分でも応用した使い方を考えられそうだった。  

# 今後どう活かすのか
`ghq`コマンドを最新にして、`zshrc`ファイルも整理したので、前よりもさらにソースコードリーディングや開発がラクになった。  
2020年はもっとコーディングしていくぞ。

ちなみに、[@songmu](https://twitter.com/songmu)さんはご自分のブログでもよく`ghq`コマンドのtipsをまとめてられているので、そちらも読んでみるとよいだろう。

- [ghqを使っていても(使っていなくても)goimportsを爆速にする | おそらくはそれさえも平凡な日々](https://songmu.jp/riji/entry/2018-08-29-ghq-goimports.html)
- [ghqで仕事用と趣味用でディレクトリ分けしてリポジトリ管理しやすくなりました | おそらくはそれさえも平凡な日々](https://songmu.jp/riji/entry/2019-12-28-ghq.html)
- [Go Modules時代に依存をghqで一括cloneする | おそらくはそれさえも平凡な日々](https://songmu.jp/riji/entry/2019-10-22-gomod-ghq.html)

# 参考
- https://github.com/motemen/ghq
- https://leanpub.com/ghq-handbook
- https://github.com/Songmu/ghq-handbook
- [ghqを使っていても(使っていなくても)goimportsを爆速にする | おそらくはそれさえも平凡な日々](https://songmu.jp/riji/entry/2018-08-29-ghq-goimports.html)
- [ghq v1リリースとghq-handbookのお知らせ | おそらくはそれさえも平凡な日々](https://songmu.jp/riji/entry/2020-01-05-ghq-v1.html)
- [ghqで仕事用と趣味用でディレクトリ分けしてリポジトリ管理しやすくなりました | おそらくはそれさえも平凡な日々](https://songmu.jp/riji/entry/2019-12-28-ghq.html)
- [Go Modules時代に依存をghqで一括cloneする | おそらくはそれさえも平凡な日々](https://songmu.jp/riji/entry/2019-10-22-gomod-ghq.html)

