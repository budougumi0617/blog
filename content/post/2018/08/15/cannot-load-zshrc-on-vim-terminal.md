+++
title = "Vimの:terminalコマンドでzshを開くと.zshrcが読み込まれない #vim #zsh"
date = 2018-08-15T09:16:25+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["vim", "environment"]
tags = ["vim", "zsh"]
+++

自分の`.zshrc`配置が悪かったのでメモ。
zshはログインシェル時とインタラクティブシェル時でロードされるディレクトリが異なる。

<!--more-->


# TL;DR
- zshはログインシェル時とインタラクティブシェル時でロードされるディレクトリが異なる
  - http://zsh.sourceforge.net/Doc/Release/Files.html
- Vimの`:terminal`コマンドではインタラクティブシェルモードで起動される
- インタラクティブシェルモードでは`$HOME/.zshrc`などは読まれない
- `$ZDOTDIR`内に`.zshrc`などを配置することで回避できる


# Vimの:terminalコマンドでzshを開くと.zshrcが読み込まれない
会社のMacを入れ替えたので、一通り環境を移行した。
[dotfiles](https://github.com/budougumi0617/dotfiles/)運用していたので、だいたい簡単に終わったのだが、Vimで`:terminal`コマンドを使うとzshが設定ファイルを読み込んでくれなかった。

```bash
This is the Z Shell configuration function for new users,
zsh-newuser-install.
You are seeing this message because you have no zsh startup files
(the files .zshenv, .zprofile, .zshrc, .zlogin in the directory
~/.zsh).  This function can help you with a few settings that should
make your use of the shell easier.

You can:

(q)  Quit and do nothing.  The function will be run again next time.

(0)  Exit, creating the file ~/.zsh/.zshrc containing just a comment.
     That will prevent this function being run again.

(1)  Continue to the main menu.

--- Type one of the keys in parentheses ---
```

# 原因:`$ZDOTDIR`配下に設定ファイルがなかったから

どうやらzshはログインシェルとして起動したときと、コマンドから起動するインタラクティブシェルで設定ファイルの読み方が違うらしい。

http://zsh.sourceforge.net/Doc/Release/Files.html

> Commands are then read from $ZDOTDIR/.zshenv. If the shell is a login shell, commands are read from /etc/zprofile and then $ZDOTDIR/.zprofile. Then, if the shell is interactive, commands are read from /etc/zshrc and then $ZDOTDIR/.zshrc. Finally, if the shell is a login shell, /etc/zlogin and $ZDOTDIR/.zlogin are read.

> If ZDOTDIR is unset, HOME is used instead. Files listed above as being in /etc may be in another directory, depending on the installation.

自分の場合は、`$ZDOTDIR`は`~/.zsh`だった（`.zshrc`内で設定している部分を消してもここになる気がする）。
たしかに`$ZDOTDIR`はそこに`.zshrc`などを配置していなかった。

# 解決方法 `$ZDOTDIR`配下に`.zshrc`へのリンクを貼る
コピペしてファイルを配置するのはメンテ性が良くないので`ln -s`コマンドで絶対パスのリンクを貼って解決した。
```
ln -s /Users/budougumi0617/.homesick/repos/dotfiles/home/.zshrc ~/.zsh/
ln -s /Users/budougumi0617/.homesick/repos/dotfiles/home/.zshenv ~/.zsh/
```

# 終わりに
いろいろアップデートしていたら、こちらの事象も起きてだいぶ時間がかかってしまった。  
なんにせよ、Vimは悪くなかったので疑って申し訳ない気持ち。

- macOS を10.13.5にアップデートしたらターミナルが死んだ
  - https://qiita.com/Kansei/items/4029a0dff197039c5e78

# 関連
- [未修正ファイルでもvim-gitgutter列を表示して差分表示で画面をガタガタさせない #vim #git](https://budougumi0617.github.io/2018/06/20/setting-vim-gitgutter-column-shows-always/)
- [[Go]vim-goとDelveでVim上からGoのデバッグを行う #vim #golang](https://budougumi0617.github.io/2018/05/30/debug-go-by-vim-go-and-delve/)
