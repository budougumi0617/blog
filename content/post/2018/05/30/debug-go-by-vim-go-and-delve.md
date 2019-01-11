+++
title= "[Go]vim-goとDelveでVim上からGoのデバッグを行う #vim #golang"
date= 2018-05-30T08:00:00+09:00
draft = false
slug = ""
categories = ["go","vim"]
tags = ["golang","vim","delve"]
author = "budougumi0617"
twitterImage = "logos/Go-Logo_Aqua.png"
+++

VimでGoを使う人は大半の人が`vim-go`プラグインをインストールしているだろう。

**fatih/vim-go**  
https://github.com/fatih/vim-go

今年の3月、ついに`vim-go`が`delve`をサポートした（他のVimプラグインを使えばすでにVimから使えたようだが）。  
VimからGoのプログラムをデバッグする方法をまとめる。

https://github.com/fatih/vim-go/blob/master/CHANGELOG.md#117---march-27-2018

```
1.17 - (March 27, 2018)
FEATURES:
Debugger support! Add integrated support for the delve debugger. Use :GoInstallBinaries to install dlv, and see :help go-debug to get started. [GH-1390]
```

なお、`vim-go`はチュートリアルを読めば大半の機能を理解することが出来るが、最近追加された機能まではチュートリアルに載っていない。

**fatih/vim-go-tutorial**  
https://github.com/fatih/vim-go-tutorial

[@hnakamur2](https://twitter.com/hnakamur2)さんが日本語翻訳版も公開してくれている。

**hnakamur/vim-go-tutorial-ja**  
https://github.com/hnakamur/vim-go-tutorial-ja

# TL;DR
- `vim-go`のデバッグ機能をヘルプからまとめた


ほぼヘルプのコマンドを動かした通りなので、英語に抵抗がないならば、Vimを開いて`:h go-debug`とすれば良い。  
見つからない場合は`vim-go`が古い状態になっている。

https://github.com/fatih/vim-go/blob/7ac1e62fa30d84946e93a3820bb6cf6a431186ed/doc/vim-go.txt#L1872

なお、今回基本的なデバッグ用語(ブレークポイント、ステップインなど)の説明は省略する。  
以下のgif動画は以下の操作を行った際の画面を記録したものである。

- `Vim`を起動
- デバッグモードを開始
- ブレークポイントを設置
- コンテニューでデバッグを開始
- スタックを確認
- ローカル変数配列を展開
- 実行中の変数の中身をデバッグプリント
- デバッグモードを終了

![vim-debug-demo](/2018/05/delve.gif)

# 事前準備

まずは最新版のvim-goに更新しておく。`vim-go`の`delve`周りに5月に一度更新が入っているので最新版を取得しておいたほうがよい。  
dein.vimでpluginをアップデートするときは`vim`を起動して以下のコマンドを実行する。

```vim
:call dein#update()
```

`:h go-debug`と入力して`DEBUGGER`のヘルプが表示されたならば`vim-go`でデバッグが出来るようになっている。

`delve`が動くことも確認しておこう。ターミナルで一通り動くことを確認しておいたほうが良い。  
自分の場合、Macで使うとdelveの起動は出来てもデバッグの開始に失敗することがあった。


- [[Go]MacのVSCodeでGoのデバッグを試すと"unexpected fault address..."エラーになる](/2017/12/24/activate-delve-on-mac/)

# Vim上でのデバッグ
今回のデバッグ対象のサンプルとして次のコードを利用した。

https://github.com/budougumi0617/gopl/blob/master/ch01/ex02/echo.go

```go
package main

import (
	"fmt"
	"io"
	"os"
	"strconv"
)

var out io.Writer = os.Stdout // modified during testing

func main() {
	for index, arg := range os.Args {
		s := "[" + strconv.Itoa(index) + "] " + arg // Convert integert to string
		fmt.Fprintln(out, s)
	}
}
```

ローカルにある上記のコードをリポジトリルート上をカレントディレクトリとして`vim`で開いた。

```bash
$ cd $GOPATH/src/github.com/budougumi0617/gopl/
$ vim ch01/ex02/echo.go
```
# `:GoDebugStart` デバッグを開始する
まず、デバッグモードの起動の仕方だ。
デバッグを開始するときは以下のように`:GoDebugStart`を実行する。(testをデバッグする場合は`:GoDebugTest`)


```
:GoDebugStart [pkg] [program-args]
```

たとえば、今回の例の場合はデバッグ対象のパッケージがカレントディレクトリではないので、以下のようになる。

```
:GoDebugStart ./ch01/ex02 test test2
```

デバッグ実行時にプログラムへ引数やフラグを渡すときはパッケージ名のあとに指定する。  
ブレークポイントを指定していた場合、そこでプログラムが止まった状態でデバッグモードに移行しているはずだ。

![GoDebugStart](/2018/05/go-debug-start.png)
また、もしここでデバッグモードの起動に失敗する場合、`:messages`を実行することで起動時のログを確認することができる。


# `:GoDebugBreakpoint` | ブレークポイントを設定する

次に、ブレークポイントを設定する。なお、ブレークポイントの設置はデバッグモードに入る前に実施することも出来る。
ブレークポイントを設定したい行の上にカーソルを移動して`:GoDebugBreakpoint`を実行するか、
ブレークポイントを設定した`LINE_NUM`行目を引数にして`:GoDebugBreakpoint $LINE_NUM`と実行する。
成功すると、左端に`>`とマーキングがされる。
ブレークポイントを解除したい場合はやはりカーソルを該当行に置くか、引数で指定して`:GoDebugBreakpoint`を実行する。

# デバッガで挙動を確認する
デバッグモード起動後は一般的なデバッグのようにステップ実行することが出来る。それぞれのコマンドにはデフォルトでキーバインドも設定されている。

|コマンド|キーバインド|説明|
|---|---|---|
|`:GoDebugContinue`|F5 | ブレークポイントまでプログラムを実行する|
|`:GoDebugNext` | F10| ステップオーバー実行。プログラム未開始時は`:GoDebugContinue`相当|
|`:GoDebugStep`|F11| ステップイン実行。プログラム未開始時は`:GoDebugContinue`相当|
|`:GoDebugStepOut`|なし| ステップアウト実行|
|`:GoDebugSet`|なし|`delve`がサポートしている一部のプリミティブ型に値をセットできる|
|`:GoDebugPrint`|F6|引数に指定された変数もしくはカーソル下の変数の値を確認できる|

デバッグ中は右下のウインドウでログや出力情報を確認することができる。

![GoDebugPrint](/2018/05/go-debug-print.png)

# 変数の情報を確認する
左下のウインドウに変数情報が表示されている。配列情報はその変数にカーソルを合わせてエンターを押下すれば情報を展開することも出来る。

# スタック情報を確認する
左のウインドウはスタック情報であり、変数名と同様に、確認したいスタック名でエンターをクリックすれば、呼び出し元の情報を確認することも可能。


# `:GoDebugRestart` | デバッグを再開始する
ブレークポイントなどを不用意に通過してしまった場合はリスタートすることもできる。


# `:GoDebugStop` | デバッグを終了する
デバッグを終了する場合は`:GoDebugStop`を実行する。
デバッグ用のウインドウやブレークポイント情報がすべて取り除かれ、通常表示のVimに戻るはずだ。

# 終わりに
以上で`Vim-go`と`delve`でデバッグする方法をまとめた。
UIに関しては`delve`をそのまま使うより、圧倒的にvim-goのほうがリッチだった気がする。
ますますVimでGoを書くのが楽しくなった。

# 関連
- [[Go]MacのVSCodeでGoのデバッグを試すと"unexpected fault address..."エラーになる](/2017/12/24/activate-delve-on-mac/)
- [Goのデバッガ(Delve)のいろいろな起動のしかた(引数を渡して起動、起動中のプロセスにアタッチして起動 etc...)](/2018/04/08/debug-by-delve/)


