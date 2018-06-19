+++
title= "未修正ファイルでもvim-gitgutter列を表示して差分表示で画面をガタガタさせない #vim #git"
date= 2018-06-20T08:36:52+09:00
draft = false
slug = ""
categories = ["vim"]
tags = ["vim", "git"]
author = "budougumi0617"
+++

vim-gitgutterはvimで編集中のファイルの左端にgitの差分情報を表示してくれるプラグインだ。

https://github.com/airblade/vim-gitgutter

未修正の状態でもvim-gitgutterラインを常に表示させて画面をガタガタしない方法を調べた。「画面がガタガタする」の意味はGIF画像を見てもらえばわかると思う。

![means-gatagata](/2018/06/gitgutter.gif)

# TL;DR
- デフォルト設定だと差分がないファイルを編集したときに左にずれるのが嫌だった
- `.vimrc`に`set signcolumn=yes`を追加する

# 表示がズレるのはストレス
vim-gitgutterプラグインを使うと、vimでgitリポジトリ以下のファイルを編集しているときに、その差分が表示される。  
ただ、デフォルト設定だと未修正のファイルを開いたときvim-gitgutter用の表示列が表示されない。  
そのため、編集を始めたときに現れるvim-gitgutter用の表示列が現れると表示中の情報が左にずれる（冒頭のGIFの通り）

# 解決方法: `.vimrc`に`set signcolumn=yes`を追加する
これを防ぐには、`set signcolumn=yes`をすればよい。  
この設定をしておくと、未修正ファイルを開いたときも最初からvim-gitgutter用の表示列が表示されているので画面がズレない。

https://github.com/budougumi0617/dotfiles/blob/68683ebd5b7e3e8a930db5587692bdbefa4e6e39/home/.vimrc#L552-L553

```vimrc
" Show gitgutter column always
set signcolumn=yes
```

vimのバージョンが7.4未満の場合は代わりに`g:gitgutter_sign_column_always`を`1`とする

https://github.com/airblade/vim-gitgutter/blob/a986ab054788776dca269d6c289b470255d54e8c/doc/gitgutter.txt#L342-L351

```
                                               *g:gitgutter_sign_column_always*
Default: 0

This legacy option controls whether the sign column should always be shown, even
if there are no signs to display.

From Vim 7.4.2201, use 'signcolumn' instead:
>
    set signcolumn=yes
<

```

# 終わりに
ググり力が足りないせいか、日本語で言及されている記事がなかったのでメモしておいた。


- Vim Git Gutter
  - https://github.com/airblade/vim-gitgutter/blob/a986ab054788776dca269d6c289b470255d54e8c/doc/gitgutter.txt#L342-L351
- How can I make the Sign Column show up all the time even if no Signs have been added to it?
  - https://superuser.com/questions/558876/how-can-i-make-the-sign-column-show-up-all-the-time-even-if-no-signs-have-been-a
- Vimメモ : vim-gitgutterで差分を左端に表示する
  - https://wonderwall.hatenablog.com/entry/2016/03/26/211710

