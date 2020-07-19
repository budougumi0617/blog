+++
title= "[Go] gopls 0.4.3で構造体を初期化（\"fillstruct\"）しようとしても、\"No code actions found\"とだけ表示される"
date= 2020-07-18T13:50:01+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["vim","go"]
tags = ["vim","lsp","gopls","golang","vim-lsp"]
keywords = ["vim","LSP","vim-lsp","gopls","fillstruct"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

gopls 0.4.3を使って構造体を初期化（`fillstruct`）しようとする（正確に言うとvim-lspで`:LspCodeAction`をする）と、`No code actions found`と表示されるだけで初期化ができなかった。  
0.4.3でも構造体の初期化を行うためのメモ。


<!--more-->

# TL;DR
- Vimでgoplsを使ったGoの開発環境構築をしている
- 構造体を初期化する`fillstruct`機能がローカルでは動作しなかった
    - https://twitter.com/mattn_jp/status/1278333328903426048
- gopls 0.4.3から`fillstruct`機能は無効になっているので、明示的に有効にする必要がある。
- vim-lspで有効にするには次のような設定になる。

```vimrc
  let g:lsp_settings = {}
  let g:lsp_settings['gopls'] = {
    \  'workspace_config': {
    \    'usePlaceholders': v:true,
    \    'analyses': {
    \      'fillstruct': v:true,
    \    },
    \  },
    \  'initialization_options': {
    \    'usePlaceholders': v:true,
    \    'analyses': {
    \      'fillstruct': v:true,
    \    },
    \  },
    \}
```

今回の現象が発生する各ツールのバージョンは次の通り。

```
$ vim --version
VIM - Vi IMproved 8.2 (2019 Dec 12, compiled May 19 2020 22:07:34)
macOS version
Included patches: 1-800
Compiled by Homebrew
...
$ gopls version
golang.org/x/tools/gopls 0.4.3
    golang.org/x/tools/gopls@v0.4.3 h1:irz7Q+XdHNECamFKbNWKvMV2Ak6zBbwdwbZndG4545I=
```


# goplsを使って構造体を初期化したい
Vim、vim-lsp、goplsを使ってGoの開発環境を構築している。  
vim-go利用時に便利だった構造体を中身をゼロ値で初期化する`fillstruct`機能が便利だったので、goplsでも使いたいと思った。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">そういえばあまりアナウンスされてなかったけど、最近の gopls には Fill Struct 機能が入ってます。vim-lsp からであれば :LspCodeAction で使える様になってます。 <a href="https://twitter.com/hashtag/golang?src=hash&amp;ref_src=twsrc%5Etfw">#golang</a> <a href="https://t.co/HxyBvYRrCa">pic.twitter.com/HxyBvYRrCa</a></p>&mdash; mattn (@mattn_jp) <a href="https://twitter.com/mattn_jp/status/1278333328903426048?ref_src=twsrc%5Etfw">July 1, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

ただ、実際に自分のVimでtweetの通りに（`:LspCodeAction`を）実行しても`No code actions found`と出るだけで構造体の初期化ができなかった。

goplsは`go get golang.org/x/tools/gopls@latest`して2020/07/17時点で最新のバージョン（0.4.3）を利用していた。

# gopls0.4.3以降ではfillstruct機能はデフォルト無効になっている
vim-jp Slackで質問していろいろ調べてもらった結果、goplsの`fillstrcut`機能は2020/07/09にリリースされたv0.4.3からはデフォルトオフになっていた。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/tools/releases/tag/gopls%252Fv0.4.3" data-iframely-url="//cdn.iframe.ly/CALRbZC"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

> Disable the fillstruct analysis by default.
> We recently uncovered some performance issues with the analysis, leading us to disable it by default.
> Once those issues are resolved, we will enable it by default again.

パフォーマンス問題が解決するまではデフォルト無効になったようだ。
リリースノートにはVS Codeで有効化する方法が書いてあるが、vim-lspでこれと同じ`fillstrcut`機能を有効化する設定方法は次のとおり。

```vimrc
  let g:lsp_settings = {}
  let g:lsp_settings['gopls'] = {
    \  'workspace_config': {
    \    'usePlaceholders': v:true,
    \    'analyses': {
    \      'fillstruct': v:true,
    \    },
    \  },
    \  'initialization_options': {
    \    'usePlaceholders': v:true,
    \    'analyses': {
    \      'fillstruct': v:true,
    \    },
    \  },
    \}
```

## Tips
調査の中で教えてもらった便利なことをメモしておく
### 特定のバージョンのgoplsを利用する
再現などの用途で特定のバージョンのライブラリを利用したいときは`go get`するときに`go get golang.org/x/tools/gopls@v0.4.2`とする。
goplsを`go get`するときは`-u`を使ってはいけない。

### LspInstallServerコマンドについて
vim-lsp-settingspの`:LspInstallServer`を行うと、独自にgoplsのバイナリがインストールされてしまうらしい。
ターミナル上で手動`go get`を使って取得したgoplsを使いたいときは、`:LspUninstallServer gopls`しておくこと。  
また、vim-lsp-settingsでインストールしたバイナリを更新したいときは、再度`:LspInstallServer gopls`をすればVimが見ているgoplsが更新される。

### LspCodeLensで使える機能はまだVimには早いかも
このあたりの設定を見ると、LSPから`go test`が実行できそうに見える。
https://github.com/golang/tools/blob/0dca43fff13b95e02f929f8c7fd7e5952943e224/internal/lsp/source/options.go#L103-L109

```go
SupportedCommands: []string{
  CommandTest,
  CommandTidy,
  CommandUpgradeDependency,
  CommandGenerate,
  CommandRegenerateCgo,
},
```

これは次のような設定を使うと、有効化される。
```vimrc
\ "codelens": {
\   "test": v:true
\ },
```
ただ、vim-lspにはLSPが実行したテストの結果をパースする機能がないため、UI上でその結果を得られないためまだちゃんと使える機能ではないらしい。


# 終わりに
リリースノートを読んでから使おうね案件だった。  
ただ、自分でリリースノートを見ても、vim上で`"gopls": {"analyses": {"fillstruct": true}}`の設定をどう書けばいいのかわからなかったので、質問してよかった。  
ローカルで再現してくれたり、`g:lsp_settings`の書き方を教えてくださったり、vim-jp Slackのみなさんに感謝！！

（`fillstruct`はまだ載っていないが、）goplsでできることは次のMarkdownにまとまっているので、他にも有効化すべき機能はないか探してみる。
https://github.com/golang/tools/blob/master/gopls/doc/analyzers.md

vim-goからLSPに移行をしてみて、だいぶ設定ができたと思うので、もうちょっとこなれたら改めてLSPの設定をまとめようと思う。
- https://github.com/budougumi0617/dotfiles/blob/master/home/.vimrc
- https://github.com/budougumi0617/dotfiles/blob/master/home/.vim/dein.toml

# 参考
- https://twitter.com/mattn_jp/status/1278333328903426048
- https://github.com/golang/tools/releases/tag/gopls%2Fv0.4.3
- https://github.com/golang/tools/blob/master/gopls/doc/analyzers.md
- https://github.com/mattn/vim-lsp-settings
- https://github.com/prabirshrestha/vim-lsp