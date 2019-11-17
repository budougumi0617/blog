+++
title= "[Go] \"vim-go: [golangci-lint] FAIL\"エラーが出てvim-goでgolangci-lintが動かない"
date= 2019-11-18T05:31:03+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go", "Vim"]
tags = ["golang", "vim", "golangci-lint"]
keywords = ["Vim", "Go", "golang", "golangci-lint"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++



公私でGoを書くときはVimを使っている。  
ファイル保存時に`golangci-lint`を実行するように設定しているのだが、表題のエラーが出て静的解析が実行されなかったので原因調査・解決した。

<!--more-->

# TL;DR
- VimでGoを書いているとき、`vim-go: [golangci-lint] FAIL`エラーがでて`golangci-lint`が失敗するようになった
- `:messages`を見てもわからなかった
- `:GoMetaLinter`コマンドを実行したところ、本当のエラーメッセージがわかった
    - `level=error msg="Running error: --enable-all and --disable-all options must not be combined"`
- `.golangci.yml`の設定を修正して解決した

# エラーが出てvim-goでgolangci-lintが動かない

公私でGoを書くときはVimを使っている。


<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golangci/golangci-lint" data-iframely-url="//cdn.iframe.ly/DL8d5gb?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/fatih/vim-go" data-iframely-url="//cdn.iframe.ly/Yf6623U?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


私のVimでは（`golangci-lint`を使う設定をして、）`GoMetaLinterAutoSaveToggle`オプションを有効にしているので、ファイル保存時に`golangci-lint`が実行される。

- GoMetaLinterAutoSaveToggle | fatih/vim-go
    - https://github.com/fatih/vim-go/blob/56edf0c53dca497d3caea36d101b0feaf73e97e0/doc/vim-go.txt#L830

ここで、あるGoのプロジェクトで作業しているときだけ、ファイル保存時に`vim-go: [golangci-lint] FAIL`とエラーメッセージが表示されるだけで静的解析が実行されなくなった。

# 原因調査： `:GoMetaLinter`コマンドを直接実行してみる
コマンドラインから直接`golangci-lint`コマンドを実行しても正常に実行できていた。  
`:messages`コマンドでVimが出力しているログを見ても、何が問題で`golangci-lint`コマンドが失敗しているのかよくわからなかった。

- message - Vim日本語ドキュメント
    - https://vim-jp.org/vimdoc-ja/message.html#:messages

起動直後にファイルを保存してエラーが出た状態で`:messages`コマンドでログをみても以下の通り。

```
Messages maintainer: Bram Moolenaar <Bram@vim.org>
"main.go" 100L, 2767C
vim-go: initializing gopls
"main.go" 100L, 2767C written
vim-go: [golangci-lint] dispatched
vim-go: [golangci-lint] FAIL
```

ここで、`vim-go`を利用しているVimでは、`:GoMetaLinter`コマンドを使うことで、明示的に`golangci-lint`を実行できる。

- `:GoMetaLinter` | fatih/vim-go
    - https://github.com/fatih/vim-go/blob/56edf0c53dca497d3caea36d101b0feaf73e97e0/doc/vim-go.txt#L651

`:GoMetaLinter`コマンドを実行してみたところ、以下のようなエラーメッセージを確認することができた。

```
level=error msg="Running error: --enable-all and --disable-all options must not be combined"

```

どうやら、プロジェクトに配置してある`.golangci.yml`に記載されている設定と、`vim-go`で`golangci-lint`コマンドを呼び出すときのオプションが競合しているのが原因だった。
`vim-go`は`golangci-lint`コマンドを呼び出すとき以下のように実行コマンドを組み立てる。この中で、`--disable-all`オプションを指定している。

- https://github.com/fatih/vim-go/blob/75fa6fe70c5b9d7f7856bc26972d9ee97d99f7db/autoload/go/lint.vim#L242-L254

```vim
function! s:golangcilintcmd(bin_path)
  let cmd = [a:bin_path]
  let cmd += ["run"]
  let cmd += ["--print-issued-lines=false"]
  let cmd += ['--build-tags', go#config#BuildTags()]
  let cmd += ["--disable-all"]
  " do not use the default exclude patterns, because doing so causes golint
  " problems about missing doc strings to be ignored and other things that
  " golint identifies.
  let cmd += ["--exclude-use-default=false"]

  return cmd
endfunction
```

今回、プロジェクトで設置してある`.golangci.yml`ファイルでは`enable-all`をベースに静的解析の設定を組み立てていため、`enable-all`と`disable-all`設定が競合していた。

```yaml
linters:
  enable-all: true
  disable:
    - funlen
    - dogsled
```

# 解決方法：`.golangci.yml`の設定を修正する
今回は以下の理由により、`.golangci.yml`側の設定方法を修正することで対応した。

- Vim側の`golangci-lint`コマンドの呼び出し時の設定を変更する方法がよくわからなかった
- `enable-all`ベースで設定していると、`golangci-lint`コマンドが新しいツールをサポートしたとき、意図せず有効化される
- `disable-all`したあと、明示的に利用する静的解析ツールを指定したほうがヒューマンリーダブルな設定になる


`golangci-lint`コマンドは`golangci-lint linters`コマンドでそのプロジェクト上（ディレクトリ上）の静的解析の設定を確認することができる。
なので、一度`enable-all`ベースの設定で`golangci-lint linters`コマンドを実行し、ファイルに出力を保存しておいた。

```bash
$ golangci-lint linters
Enabled by your configuration linters:
bodyclose: checks whether HTTP response body is closed successfully [fast: true, auto-fix: false]
deadcode: Finds unused code [fast: true, auto-fix: false]
...

Disabled by your configuration linters:
depguard: Go linter that checks if package imports are in a list of acceptable packages [fast: true, auto-fix: false]
dogsled: Checks assignments with too many blank identifiers (e.g. x, _, _, _, := f()) [fast: true, auto-fix: false]
...

```

その後、`disable-all`ベースの設定に書き直し、`golangci-lint linters`コマンドの結果が以前と同じであることを確認した。

# 終わりに
私のVimの設定は一応`golangci-lint`や`gopls`を利用するように設定しているのだが、一度整理したほうがよいかなと思いつつズルズル使っている。
[LSP系のプラグイン経由で`gopls`を実行するほうがいろいろ設定できるらしい][gopls]ので、一度整理しようかなあ。

[gopls]: https://mattn.kaoriya.net/software/lang/c/20191112100330.htm

# 参考
- https://github.com/golangci/golangci-lint
- https://github.com/fatih/vim-go
- [Go 言語の Language Server「gopls」が completeUnimported に対応した。](https://mattn.kaoriya.net/software/lang/c/20191112100330.htm)
