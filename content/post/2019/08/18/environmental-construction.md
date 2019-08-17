+++
title= "新しいMacに開発環境を構築（移行）したときに行なうこと"
date= 2019-08-18T07:31:53+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["dotfiles"]
tags = ["dotfiles", "git", "brew", "macOS"]
keywords = ["環境構築", "dotfiles", "macOS", "Mac"]
twitterImage = "twittercard.png"
+++

最近開発用のMacの入れ替えを行なったので、新しいPCを使うときに行なう環境構築のメモ。
ブログ記事を書くときは一般的な内容になるよう心がけているが、今回は自分用のメモになっている。

<!--more-->

# TL;DR
- `dotfiles`リポジトリを作っておくと便利
    - 私は加えて`homeshick`を使っている
    - https://github.com/andsens/homeshick
    - VimやZshの設定はこれでひきつぐ
    - 会社用のgitの設定は別ファイルにしておいて`.gitconfig`ファイル内で`include`を使ってロードするようにしておく
    - 機密情報を含んだ環境変数も別ファイルにして`source`で読み込むようにしておく
- Brewでインストールしたアプリや設定は`brew bundle`で引き継げる
- VS Codeの設定の引き継ぎはCode Settings Syncを使う
    - https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync
- MacOSの設定もApple Scriptにしておいたほうが良い。
    - caps lockを変更する、キーリピートを最速にする、など

なお、個別のプラグインなどの紹介などはしていない。私の開発用の設定は以下のリポジトリにある。

- https://github.com/budougumi0617/dotfiles

# dotofileの設定
各種設定ファイルはGitHub上で管理している。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/dotfiles" data-iframely-url="//cdn.iframe.ly/1nz9RRz"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`homeshick`というCLIツールを使うと簡単にホームディレクリに`symlink`ができるのでラクだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/andsens/homeshick" data-iframely-url="//cdn.iframe.ly/xi21H7X"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

以下の流れで設定が終わる。

```bash
git clone https://github.com/andsens/homeshick.git $HOME/.homesick/repos/homeshick
source $HOME/.bashrc
homeshick clone https://github.com/budougumi0617/dotfiles.git
```

## 環境変数の設定
ドットファイルはGitHubで管理する…と言っても、GitHubにアップロードできないトークン情報などを環境変数に書いて`export`したりしていると思う。
私の場合そういう環境変数は`.bashrc`に記載している。`.zshrc`の中で`source`を使って`.bashrc`を読み込む（ファイル名が`.bashrc`の必要はないのだけれど）。

```bash
$ cat ~/.zshrc
...
source ~/.bashrc
...
```

## gitconfigの設定
会社のリポジトリに個人のメールアドレスでコミットしたくなかったり、会社で使うときだけ必要な設定がある。
`gitconfig`は`include`という別のファイルの設定を`import`できる設定があるので、これを最後に書いておく。

- https://git-scm.com/docs/git-config#_includes

```
$ cat ~/.gitconfig
[user]
  name = budougumi0617
...
[include]
  path = .gitconfig.local
```

私の場合は、`.gitconfig.local`ファイルがホームディレクトリにあればそこに書いてある設定で諸々の設定を上書きするようにしてある。
最近は`includeif`もあるらしいので、好きなほうを使えば良いと思う。

- [gitconfigのincludeが便利](https://qiita.com/knt45/items/51b8a8645f36fb0a6d01)
- [[git] gitconfigで会社用アカウントと個人用アカウントを楽に使い分けする](https://qiita.com/Ancient_Scapes/items/64f239f89d25a3b9f520)

# Zshの設定
プラグインマネージャを使って`.zshrc`を書いたら`dotfiles`リポジトリに保存しておく。
私は`zplugin`を使うようにした。新しいPCで`dotfiles`を`clone`、ホームディレクトリに`.zshrc`を置いたら、`zplugin`をインストールし、デフォルトシェルを`zsh`に変えておく（`macOS`のデフォルトシェルはそのうちZshになるらしいが）

```bash
# https://github.com/zdharma/zplugin#installation
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/zdharma/zplugin/master/doc/install.sh)"
$ chsh -s /usr/local/bin/zsh

```

あとは次回ターミナル起動時に各種プラグインが自動でダウンロードされる。

ターミナルの表示をいじっているので、`SourceCodePro+Powerline+Awesome+Regular.ttf`をインストールしないと文字化けする。

- [Falkor/dotfiles/fonts/SourceCodePro+Powerline+Awesome+Regular.ttf](https://github.com/Falkor/dotfiles/blob/master/fonts/SourceCodePro%2BPowerline%2BAwesome%2BRegular.ttf)


通常のPowerlineフォントはここにある。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/powerline/fonts" data-iframely-url="//cdn.iframe.ly/TebRSQf"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# Vimの設定
`dein.vim`でプラグイン管理しているので、Vimの初回起動時にもろもろインストールされる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/Shougo/dein.vim" data-iframely-url="//cdn.iframe.ly/QjBrr1T"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

一部のプラグインが依存しているpythonのプラグインだけ手動でインストールする必要がある。

```bash
$ pip3 install pynvim
```

# VS Codeの設定
VS Codeの設定は`Sync`プラグインを使ってGist経由で共有できる。
予め古いPCで「Upload Your Settings」をしておく。
プラグインインストール時に古いPCで連携していたGistを選択して「Download your Settings」を行えば、他のインストールしていたプラグインやスニペットなどの設定が全て引き継がれる。


- Settings Sync | VisualStudio Marketplace
- https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync

# Brewで管理しているアプリ
Mac利用者はHomeBrewでアプリを管理していることが多いと思う。HomeBrewは`.Brewfile`にインストールしたアプリリストを保存して新しいPCに移行できる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://qiita.com/vochicong/items/f20afc89a6847cd58f0f" data-iframely-url="//cdn.iframe.ly/LVRBYFg"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`fzf`、`peco`、`jq`、`tmux`などの便利ツールはこのステップでインストールが終わる。
HomeBrew経由でインストールしていれば、GUIアプリもこれでインストールが完了する。

# macOSの設定
Apple Scriptを使えば各種設定も自動化できるらしいのだが、私はまだそこまでできていないので、もろもろ手動で設定している。
私がやる設定変更は以下

- `xcode-select --install`
    - お作法…
- ディスプレイを増やしてショートカット割当をする
    - [macOSでディスプレイ1枚で作業する技術](https://qiita.com/saboyutaka/items/d6cfd2a2b60f1a374d60)
    - 便利なので知ってからずっとやっている。
- `caps lock`を`ctrl`に変更する
    - [Mac OS XでCaps Lockキーを無効化したり、他の修飾キーに入れ替えたりする方法][caps]
    - caps lockがなくて困ったことが一度もないのに、なんであんないい位置にあるのだろうか？
- `⌘英かな`のインストール
    - https://ei-kana.appspot.com/
    - USキーボードなので。macOSの設定でcaps lockがいじれるのでKarabinerは止めてしまった。
- `mini calendar`のインストール
    - <div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://apps.apple.com/us/app/mini-calendar/id1088779979?mt=12" data-iframely-url="//cdn.iframe.ly/CRI7zJQ?media=0"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
    - [Mac のメニューバーやドックからカレンダーを確認できる Mini Calendar][minical]
    - さっとカレンダー見れて便利
    - メニューバーの並びは`command`キーをクリックしながらドラッグアンドドロップで変更できる
- `Clipy`のインストール
    - https://clipy-app.com/
- Touch BarからSiriを消す
    - システム環境設定→キーボード→Touch Barをカスタマイズ
    - 初期配置だと`delete`をクリックするときにたまに誤タッチしてめんどくさい
- キーリピートなどを最速にする
    - これしないとVimが不便

[caps]: http://inforati.jp/apple/mac-tips-techniques/system-hints/how-to-disable-caps-lock-key-of-mac.html
[minical]: https://loumo.jp/wp/archive/20160809120027/

# clipyの設定
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="http://clipy-app.com/" data-iframely-url="//cdn.iframe.ly/Ey4fpUS"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

クリップボードにバッファをもたせたり、スニペットを登録できるので、clipyというソフトを使っている。
調査に使うSQLクエリや、JIRAなどに記載するお決まり文句などをスニペットに登録しておくと便利だ。
古いPCのClipyの「スニペットの編集」画面からスニペットをXMLでエクスポートして引き継げる。

# iTunesでやっておくこと
iTunesは1アカウントで利用できるデバイス数に制限がある。古いPCを処分する前に連携を解除するのを忘れないこと。

移行前のPCでiTunesのメニューから「アカウント」→「認証」→「このPCの認証を解除」


# 終わりに
`dotfiles`があったので比較的ラクに環境構築が終わった。  
今回、フォントインストールを間違えていてずっとiTerm2の文字化けにハマっていたので、書き起こしておくことにした。  
ただ、Macの設定は全部手動でやったので、Apple Scriptに落とし込んでおいたほうが良さそうだ。  
自分用のメモだがどなたかの参考になれば幸い。また、「これはこうすればもっと簡単に自動化できるよ！」というのをご存知の方がいらっしゃったら教えていただけると嬉しい。

# 参考
- [最強の dotfiles 駆動開発と GitHub で管理する運用方法](https://qiita.com/b4b4r07/items/b70178e021bef12cd4a2)
- https://github.com/zdharma/zplugin
- https://github.com/Shougo/dein.vim
- https://github.com/andsens/homeshick
- https://github.com/Homebrew/homebrew-bundle
- https://github.com/powerline/fonts
- [Falkor/dotfiles/fonts/SourceCodePro+Powerline+Awesome+Regular.ttf](https://github.com/Falkor/dotfiles/blob/master/fonts/SourceCodePro%2BPowerline%2BAwesome%2BRegular.ttf)
- [Mini Calendar](https://apps.apple.com/us/app/mini-calendar/id1088779979)
- [Clipy](https://clipy-app.com/)
- [⌘英かな](https://ei-kana.appspot.com/)
- [brew bundleでMacのアプリをまとめてインストール・管理](https://qiita.com/vochicong/items/f20afc89a6847cd58f0f)
- [macOSでディスプレイ1枚で作業する技術](https://qiita.com/saboyutaka/items/d6cfd2a2b60f1a374d60)
