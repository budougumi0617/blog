+++
title= "[Vim] vim-goを使わず、LSPを使ってGoの開発環境を構築する"
date= 2020-07-24T07:00:26+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["vim","go"]
tags = ["vim","lsp","gopls","golang","vim-lsp"]
keywords = ["vim","LSP","vim-lsp","gopls","fillstruct"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

2020年にもなったので、vim-goを卒業して、vim-lsp（gopls）を使ったVimの開発環境を構築する。

<!--more-->

# TL;DR
- vim-goを卒業してgoplsとvim-lspを使った開発環境を構築する
- VimでLSP（とその他プラグイン）を使えば以下のことができる
    - リアルタイムで静的解析の結果をエディタ上に反映する
    - ポップアップで静的解析のエラーを表示する
    - ポップアップで関数定義などのコメントを表示する
    - 定義元へジャンプができる。
    - `package名.`などを入力IDEのような補完候補が表示さえる
    - `func`と入力してタブを押下するとスニペットが展開される。
    - `&http.Client{}`と書いたあと`:LspCodeAction`で構造体のフィールドをゼロ値で初期化する
    - `import`をよしなに解決する（`goimport`）
    - `:w`による自動ソースコード整形、およびそのエラー表示
    - Vim上からテストを実行する
    - Vim上からdelveを使ってデバッグを行う
- 設定方法などをまとめた

実行環境は次の通り。

```bash
$ vim --version
VIM - Vi IMproved 8.2 (2019 Dec 12, compiled May 19 2020 22:07:34)
macOS version
Included patches: 1-800
$ gopls version
golang.org/x/tools/gopls 0.4.3
    golang.org/x/tools/gopls@v0.4.3 h1:irz7Q+XdHNECamFKbNWKvMV2Ak6zBbwdwbZndG4545I=
```

ひとまず動かしてみたいならば、以下のファイルを利用すれば同様の環境が構築できる（一部pythonライブラリなどの依存解決が必要）。
https://github.com/budougumi0617/dotfiles/blob/9551a0c32ed199b6d3c0f53a71dda4949163beb2/home/.vimrc
https://github.com/budougumi0617/dotfiles/blob/9551a0c32ed199b6d3c0f53a71dda4949163beb2/home/.vim/dein.toml

静的解析の結果を表示したり、
![静的解析結果を表示](/2020/07/24_show_warnings.png)

補完を出したり十分開発に耐えうると思う。
![補完](/2020/07/24_comp.png)

# vim-goは？
タイトルの通り、vim-goは今回使わないことにした。

vim-goは少し前なら「vim-go入れればVimのGoの開発環境構築はおしまい！」みたいなプラグインだった。
しかし、vim-go自体が少し微妙な立ち位置になってきたり、「vim-go経由でLSP使うならばもうLSPを直接使えばいいのでは？」というのが2020年07月現在のステータスだと思っている。

詳しい経緯はこのへんの記事が詳しい。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://mattn.kaoriya.net/software/lang/go/20181217000056.htm" data-iframely-url="//cdn.iframe.ly/1j5lwRT"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://arslan.io/2018/10/09/taking-an-indefinite-sabbatical-from-my-projects/" data-iframely-url="//cdn.iframe.ly/bAcYoLS?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

なので、今回はゼロからvim-goを使わないVimのGoの開発環境構築を行なった。

# プラグインマネージャの設定
まずはプラグインの自動インストール・アップデートをするためにプラグインマネージャを導入する。
なんとなく通ぶってdein.vimを使うことにした。

dein.vimのインストール設定は[@gorilla][gorilla0513]さんの記事に記載されている設定をほぼそのまま使う。

[gorilla0513]: https://twitter.com/gorilla0513
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://knowledge.sakura.ad.jp/23248/" data-iframely-url="//cdn.iframe.ly/ilkLdx2"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

次の設定をvimrcファイルに記載する。

```vimrc
let s:dein_dir = expand('~/.cache/dein')
let s:dein_repo_dir = s:dein_dir . '/repos/github.com/Shougo/dein.vim'

if &runtimepath !~# '/dein.vim'
  if !isdirectory(s:dein_repo_dir)
    execute '!git clone https://github.com/Shougo/dein.vim' s:dein_repo_dir
  endif
  execute 'set runtimepath^=' . s:dein_repo_dir
endif

if dein#load_state(s:dein_dir)
  call dein#begin(s:dein_dir)

  if !has('nvim')
    call dein#add('roxma/nvim-yarp')
    call dein#add('roxma/vim-hug-neovim-rpc')
    " for vim-delve
    call dein#add('Shougo/vimshell.vim')
    call dein#add('Shougo/vimproc.vim', {'build' : 'make'})
  endif

  let s:rc_dir = expand('~/.vim')
  if !isdirectory(s:rc_dir)
    call mkdir(s:rc_dir, 'p')
  endif
  let s:toml = s:rc_dir . '/dein.toml'

  call dein#load_toml(s:toml, {'lazy': 0})

  call dein#end()
  call dein#save_state()
endif

if dein#check_install()
  call dein#install()
endif

let s:removed_plugins = dein#check_clean()
if len(s:removed_plugins) > 0
  call map(s:removed_plugins, "delete(v:val, 'rf')")
  call dein#recache_runtimepath()
endif
```

`if !has('nvim')`の節は通常のvimでは一部追加のプラグインを入れないと後述のプラグインが動かないため。
また、pythonのモジュールにも依存しているため、次のコマンドでインストールを済ませておくこと。

```bash
$ pip3 install --user pynvim
$ pip3 install --user neovim
```
あとは`~/.vim/dein.toml`にプラグインの依存を書き始めることができる。依存プラグインはVim起動時に自動インストールされる。

ただ、dein.vimを使っているが、がっつりdein.vimの使いかたを覚えるわけでもないので、vim-plugでもよかったかもしれない。
ソフトウェアデザイン8月号のコラムによると、vim-plugのほうがお手軽のようだ。

- https://github.com/junegunn/vim-plug


# LSPの設定
何はさておきLSPの設定をする。
先ほどの設定ならば、`~/.vim/dein.toml`を読み込むようになっているのでTOMLファイルを作成し、次の依存を書く。

```toml
[[plugins]]
repo = 'prabirshrestha/async.vim'

[[plugins]]
repo = 'prabirshrestha/asyncomplete.vim'

[[plugins]]
repo = 'prabirshrestha/asyncomplete-lsp.vim'

[[plugins]]
repo = 'prabirshrestha/vim-lsp'

[[plugins]]
repo = 'mattn/vim-lsp-settings'

[[plugins]]
repo = 'SirVer/ultisnips'

[[plugins]]
repo = 'honza/vim-snippets'

[[plugins]]
repo = 'thomasfaingnaert/vim-lsp-snippets'

[[plugins]]
repo = 'thomasfaingnaert/vim-lsp-ultisnips'

[[plugins]]
repo = 'hrsh7th/vim-vsnip'

[[plugins]]
repo = 'hrsh7th/vim-vsnip-integ'
```

GoのLSPサーバとしては次の2つを利用する。

- https://github.com/golang/tools/tree/master/gopls
- https://github.com/nametake/golangci-lint-langserver

goplsはGoチームがメンテしている公式LSPだ。これを使うと補完や定義元ジャンプなどができるようになる。
加えてgolangci-lint-langserverを利用すればリアルタイムで静的解析結果をVim上で確認できる。

あとはvimrcに次のような設定をかく。これはだいたいmattnさんの設定を参考にした。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://mattn.kaoriya.net/software/vim/20191231213507.htm" data-iframely-url="//cdn.iframe.ly/83rbwG4"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

```vimrc
function! s:on_lsp_buffer_enabled() abort
  setlocal omnifunc=lsp#complete
  setlocal signcolumn=yes
  nmap <buffer> gd <plug>(lsp-definition)
  nmap <buffer> <C-]> <plug>(lsp-definition)
  nmap <buffer> <f2> <plug>(lsp-rename)
  nmap <buffer> <Leader>d <plug>(lsp-type-definition)
  nmap <buffer> <Leader>r <plug>(lsp-references)
  nmap <buffer> <Leader>i <plug>(lsp-implementation)
  inoremap <expr> <cr> pumvisible() ? "\<c-y>\<cr>" : "\<cr>"
endfunction

augroup lsp_install
  au!
  autocmd User lsp_buffer_enabled call s:on_lsp_buffer_enabled()
augroup END
command! LspDebug let lsp_log_verbose=1 | let lsp_log_file = expand('~/lsp.log')

let g:lsp_diagnostics_enabled = 1
let g:lsp_diagnostics_echo_cursor = 1
" let g:asyncomplete_auto_popup = 1
" let g:asyncomplete_auto_completeopt = 0
let g:asyncomplete_popup_delay = 200
let g:lsp_text_edit_enabled = 1
let g:lsp_preview_float = 1
let g:lsp_diagnostics_float_cursor = 1
let g:lsp_settings_filetype_go = ['gopls', 'golangci-lint-langserver']

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

" For snippets
let g:UltiSnipsExpandTrigger="<tab>"
let g:UltiSnipsJumpForwardTrigger="<tab>"
let g:UltiSnipsJumpBackwardTrigger="<s-tab>"

set completeopt+=menuone
```

上記の設定をして、`mattn/vim-lsp-settings`を使っていれば、LSPのインストールは簡単だ。
適当なGoのファイルをvimで開いて、`:LspInstallServer`をすればよしなにインストールされる・
これでひとまずいい感じにGoが書けるようになる。主要なところを言うと次のような操作ができるようになる。

- リアルタイムで静的解析の結果をエディタ上に反映する
- ポップアップで静的解析のエラーを表示する
- ポップアップで関数定義などのコメントを表示する
- 定義元へジャンプができる。
- `package名.`などを入力IDEのような補完候補が表示さえる
- `func`と入力してタブを押下するとスニペットが展開される。
- `&http.Client{}`と書いたあと`:LspCodeAction`で構造体のフィールドをゼロ値で初期化する
    - [[Go] gopls 0.4.3で構造体を初期化（"fillstruct"）しようとしても、"No code actions found"とだけ表示される](/2020/07/18/use_fillstruct_of_goplus_on_vim/)
- Vim上からテストを実行する
- Vim上からdelveを使ってデバッグを行う

## ファイル保存時の自動フォーマット、goimports
次のような機能はLSPではまだ実現していないので、別のプラグインを入れる。
- `import`をよしなに解決する（`goimport`）
- `:w`による自動ソースコード整形、およびそのエラー表示

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://mattn.kaoriya.net/software/vim/20200106103137.htm" data-iframely-url="//cdn.iframe.ly/q43M9Kt"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

とくに追加の設定は必要なく、プラグインへの依存をTOMLに追加するだけだ。

```toml
[[plugins]]
repo = 'mattn/vim-goimports'
```

## テストの実行
2020年7月現在LSPからGoのテストは実行できない。
そのため、次のプラグインを導入して、vimからテストを実行できるようにする。

- https://github.com/vim-test/vim-test
- https://github.com/tpope/vim-dispatch

Vimのプラグインではないが、テスト結果が見やすくなるため、私は`gotest`を使っている。
- https://github.com/rakyll/gotest

```vimrc
let test#strategy = "dispatch"
let test#go#runner = 'gotest'

nmap <silent> t<C-n> :TestNearest<CR>
nmap <silent> t<C-f> :TestFile<CR>
nmap <silent> t<C-s> :TestSuite<CR>
nmap <silent> t<C-l> :TestLast<CR>
nmap <silent> t<C-g> :TestVisit<CR>
```

## Goのデバッグ
Vimからデバッグを実行することもできる。デバッグにはGoのデファクトである`delve`を使う。
Vim上からブレイクポイントなどを設置して、デバッグを開始できる
先ほどの`vim-test`と連携させれば、デバッガを使ってテストを実行することもできる。

- https://github.com/sebdah/vim-delve
- https://github.com/go-delve/delve

```vimrc
# tのあとにCTRL+dでテストをデバッガ経由で実行する
function! DebugNearest()
  let g:test#go#runner = 'delve'
  TestNearest
  unlet g:test#go#runner
endfunction
nmap <silent> t<C-d> :call DebugNearest()<CR>
```

# その他のプラグイン
Goに直接関係ないプラグイン関係だと、次のプラグインを導入している。

- https://github.com/junegunn/fzf.vim
    - バッファやファイルをfzf経由で検索できるようになる
- https://github.com/Shougo/denite.nvim
    - ファイルを検索したり（fzfとカブってはいる）
    - https://zaief.jp/vim/denite
- https://github.com/Shougo/defx.nvim
    - ファイルエクスプローラ
    - これ使うまでvimでディレクトリうまく作れなかった…
- https://github.com/kristijanhusak/defx-git
    - defxでgit statusを表示する
- https://github.com/tpope/vim-fugitive
- https://github.com/tpope/vim-rhubarb
    - Vim上でgit操作ができる
- https://github.com/airblade/vim-gitgutter
    - ウインドウの脇にgit diffを表示できる

## その他の設定
あとはプラグインではないが、スペルチェックなどをオンにしたりタブの設定をしている。便利。

- https://vim-jp.org/vimdoc-ja/spell.html
- https://qiita.com/wadako111/items/755e753677dd72d8036d


使っている設定の全容は次の通り。
https://github.com/budougumi0617/dotfiles/blob/9551a0c32ed199b6d3c0f53a71dda4949163beb2/home/.vimrc
https://github.com/budougumi0617/dotfiles/blob/9551a0c32ed199b6d3c0f53a71dda4949163beb2/home/.vim/dein.toml

# 終わりに
業務でPhpStrom、PyCharmを使うようになったので、Goの実装をするときもGoLandを使うことが多くなった。
ただ、最近leetcode（競プロ）をするようになって「ペライチコードならばやっぱりVimでいいよな」となったのと、メルカリさんのコーディングの実況を見て「ちゃんと設定しよう！」という気持ちになった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://mercari.connpass.com/event/181012/" data-iframely-url="//cdn.iframe.ly/KYi4RxB?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

vim-goやGoLandで多用する機能がLSPと各プラグインでほぼフォローできるということがわかったのも大きい。やっぱりVimで書いているときは楽しい。
とはいえ思考のスピードでコーディングできるほどではないので、もっとVim理解しないといけない。

# 参考
- https://github.com/budougumi0617/dotfiles/blob/9551a0c32ed199b6d3c0f53a71dda4949163beb2/home/.vimrc
- https://github.com/budougumi0617/dotfiles/blob/9551a0c32ed199b6d3c0f53a71dda4949163beb2/home/.vim/dein.toml
- https://knowledge.sakura.ad.jp/23248/
- https://mattn.kaoriya.net/software/vim/20191231213507.htm
- https://mattn.kaoriya.net/software/vim/20200106103137.htm
- https://zaief.jp/vim/denite
- https://vim-jp.org/vimdoc-ja/spell.html
- https://qiita.com/wadako111/items/755e753677dd72d8036d
- https://mercari.connpass.com/event/181012/

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B08CZ2C3NZ&linkId=80eaad639e2cdb692bb0e4147c511bec"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B0899SR52S&linkId=57224cf8e56cd1400fa931c8b0e004ad"></iframe>
