+++
title = "vim-goの自動補完が効かないときの調べ方（gocode が Error parsing input file (outer block)） #vim #go"
date = 2018-10-22T08:40:19+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["vim", "go"]
tags = ["vim", "gocode", "vim-go", "golang"]
+++

`goimports`などが更新されていたので、`vim-go`で`GoUpdateBinaries`コマンドを実行したら`vim`で自動補完が効かなくなった。  
結局はgocodeの調子が悪いことが多いので、原因の調べ方をまとめる。  
（今回の場合は `let g:go_gocode_propose_source = 0` で解決した。）

<!--more-->

# TL;DR
- vim-goの自動補完に正しい候補がでなくなった
- `gocode`を`gocode exit`したあと`gocode -debug -s`で再起動する
- vimも再起動して適当なGoファイルをいじっていると、`gocode`のデバッグ出力が確認できる
- 今回は`vimrc`ファイルに`let g:go_gocode_propose_source = 0`を加えることで解決した。

なおvimバージョンは以下。
```
VIM - Vi IMproved 8.1 (2018 May 18, compiled Aug 15 2018 08:48:39)
macOS version
```

`gocode`は以下のコミットハッシュの状態([#71](https://github.com/mdempsky/gocode/pull/71)がマージされた状態)
https://github.com/mdempsky/gocode/tree/22f3bf7a9256a30885d7cd46da4657cc878f3f4f

# vim-goの自動補完が効かなくなった
`goimports`や`golint`が更新されていたので、更新しようとした。  
私はひとつひとつ更新するのが面倒なので、vimで`GoUpdateBinaries`コマンドを実行することで全て更新している。  
今回、コマンド実行後`vim-go`で自動補完が動かなくなってしまった。  
`vim-go`を更新すれば直るか？と思い(自分は`dein`を使っているので、)`call dein#update()`してみたがそれでもダメだった。

# gocodeの挙動を確認する
2018/10/20時点の`vim-go`は内部で`gocode`を利用することで自動補完を実施している。  
`gocode exit`してバックプロセスを再起動(vimでGo触ってると再度起動する)しても直らなかった。

vim自体がロギングしているログは`messages`で確認することができるが、`messages`では`gocode`が出力しているログは確認できない。  
なので、`gocode`をデバッグモードで起動して確認した。`gocode`は`gocode exit`で一度終了したあと、  
`gocode -debug -s`と実行することでデバッグ出力をターミナルに出力する状態で起動できる。  
あとはこの状態でvimを操作するとgocodeからデバッグログが出力される。

今回は以下のようなデバッグ出力が出力された。

```
$ gocode -debug -s
...
2018/10/20 16:25:27 Error parsing input file (outer block):
2018/10/20 16:25:27  /Users/budougumi0617/go/src/github.com/C-FO/ocean/tool/cipher/generate_key_pair.go:28:1: expected operand, found ';'
2018/10/20 16:25:27  /Users/budougumi0617/go/src/github.com/C-FO/ocean/tool/cipher/generate_key_pair.go:36:3: expected ')', found 'EOF'
2018/10/20 16:25:27  /Users/budougumi0617/go/src/github.com/C-FO/ocean/tool/cipher/generate_key_pair.go:36:3: expected ';', found 'EOF'
2018/10/20 16:25:27  /Users/budougumi0617/go/src/github.com/C-FO/ocean/tool/cipher/generate_key_pair.go:36:3: expected ';', found 'EOF'
2018/10/20 16:25:27  /Users/budougumi0617/go/src/github.com/C-FO/ocean/tool/cipher/generate_key_pair.go:36:3: expected '}', found 'EOF'
2018/10/20 16:25:27  /Users/budougumi0617/go/src/github.com/C-FO/ocean/tool/cipher/generate_key_pair.go:36:3: missing ',' in argument list
```

ログを頼ったり、`gocode`の更新を確認したところ`source`オプションの有無で挙動が変わるらしく、以下のオプションを`vimrc`に書いたところ直った。

```vimrc
let g:go_gocode_propose_source = 0
```

今回の原因は`gocode`でこの変更が入ったからのようだ。

- cache packages to improve speed
  - https://github.com/mdempsky/gocode/pull/71


# 終わりに
おそらく`vim-go`側も対応入ると思うので、しばらくは`vim-go`と`gocode`をこまめに更新したほうが良いのかもしれない。  
直すまで半日くらいVSCodeで作業していたのだが、VSCode(+vimプラグイン)でGo書くのもなかなか良かった。  
けどやっぱりvimのほうがシンプルで好きだ。


