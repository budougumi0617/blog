+++
title= "[vim-go]GoFillStructで構造体のフィールド初期化文を自動生成する"
date= 2018-05-06T16:25:33+09:00
draft = false
slug = ""
categories = ["go","vim"]
tags = ["golang","vim", "vim-go"]
author = "budougumi0617"
+++

Goで構造体の初期化をするときに「フィールド名なんだっけ？」となることがある。
また、フィールド名を手打ちしてタイポすることもあるだろう。  
vim-goを使っている場合、`:GoFillStruct`コマンドで構造体のフィールド初期化を簡単に行うことができる。


![フィールドを初期化する](/2018/05/fillstruct.gif)

# TL;DR
- vim-goの`:GoFillStruct`コマンドを使うと、構造体のフィールド初期化を簡単にできる
- `vim-go`のバージョンを1.15以上にする
-  構造体の初期化宣言の上にカーソルを合わせて`:GoFillStruct`コマンドを実行する


# `vim-go`のバージョンを1.15以上にする
まず`GoFillStruct`コマンドを使うにはvim-goのバージョンが1.15以上である必要がある。

https://github.com/fatih/vim-go/blob/7491209072ed4aa746e6fe7894f976ecd251801e/CHANGELOG.md#115---october-3-2017

```
FEATURES:
Add :GoFillStruct to fill a struct with all fields; uses fillstruct [GH-1443].
```

2017/10以前にvim-goをインストールしたままの場合はプラグインのアップデートを行う。  
自分の場合は`dein.vim`を使っているので、以下のコマンドでプラグインのアップデートを行なった。

```
:call dein#update()
```

#  構造体の初期化宣言の上にカーソルを合わせて`:GoFillStruct`コマンドを実行する
https://github.com/fatih/vim-go/blob/af32090927dd9ab92ba150f9b43a694338606200/doc/vim-go.txt#L837-L855

```
 :GoFillStruct
    Use `fillstruct` to fill a struct literal with default values. Existing
    values (if any) are preserved. The cursor must be on the struct you wish
    to fill.
```


vim-goのバージョンが問題ないならば、あとはそのまま利用することができる。  
適当に初期化宣言を書いたあとにvimをノーマルモードにして`GoFillStruct`コマンドを実行する。  
カーソルの位置がだいたい構造体の初期化宣言部分に合っていれば全てのフィールドの初期化文がデフォルト値で生成される。  
既に一部フィールドの初期化宣言を記載済みでも利用することができる。

実際の動きをキャプチャしたものが以下になる。
`Foo`構造体と`http.Client`構造体のフィールド初期化を`GoFillStruct`コマンドで行なった。

![フィールドを初期化する](/2018/05/fillstruct.gif)

```go
// Foo has two fields.
type Foo struct {
    StringField string
    IntField    int
}

// Before
func main() {
    foo := &Foo{StringField: "test"}
    c := &http.Client{}
}

// After GoFillStruct
func main() {
    foo := &Foo{
        StringField: "test",
        IntField:    0,
    }
    c := &http.Client{
        Transport:     nil,
        CheckRedirect: nil,
        Jar:           nil,
        Timeout:       0,
    }
}
```

# 終わりに
vim-goで出来る機能はだいたいvim-go Tutorialを読めばわかる。

https://github.com/fatih/vim-go-tutorial


`GoFillStruct`コマンドはTutorialでフォローされていないので今回記事にした。
標準パッケージの構造体でこのコマンドを使うと「あ、これもフィールドで指定できたんだ」と気づくこともあり、一度覚えたあとはかなり多用している。
