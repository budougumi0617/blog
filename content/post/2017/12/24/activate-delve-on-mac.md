+++
title= "[Go]MacのVSCodeでGoのデバッグを試すと\"unexpected fault address...\"エラーになる"
date= 2017-12-24T13:11:28+09:00
draft = false
slug = ""
categories = ["Go"]
tags = ["debug", "vscode", "golang", "delve"]
author = "budougumi0617"
+++




Macの`Visual Studio Code`(以下`VSCode`)を使って`Go`のコードの中で`Syscall`が呼ばれるまでをデバッグで確認したかったが、ステップインを使うと、`unexpected fault address...`で失敗していた。

# TL;DR
- MacのVSCodeで`Delve`でGoのデバッグを開始してステップインしようとすると、`unexpected fault address`と出る
- `Delve`を再インストールしたら直った。
	- GitHubから`derekparker/delve`を取得して`make install`でインストールし直す
- `which dlv`で古い`dlv`を見ていたら削除しておくこと

# MacにDelveを正しくインストールする。
MacでDelveを利用するには、証明書の設定が必要になる。  
`usr/bin/dlv`があったのでdelveのインストールを終えていたと思っていたのだけど、ちゃんと設定できていなかったらしい。`brew`よりも直接インストールしたほうがラクそうだったので以下の方法で再設定した。

## 1. Delveのコードを取得する

`ghq get`なり`git clone`で`delve`のソースコードを取得する。

```bash
$ ghq get git@github.com:derekparker/delve.git
または
$ git clone git@github.com:derekparker/delve.git
```
## 2. Delveをインストールする

コードからならば`make`を使うだけですぐインストールできる。

```bash
$ cd ${DELVE_REPO_DIR}
$ make install
scripts/gencert.sh || (echo "An error occurred when generating and installing a new certicate"; exit 1)
go install -ldflags="-s -X main.Build=1758..." github.com/derekparker/delve/cmd/dlv
codesign -s "dlv-cert"  /Users/budougumi0617/go/bin/dlv
```

自分の場合は`/Users/budougumi0617/go/bin/`ディレクトリにバイナリが出来た。


# 3. `which dlv`で古い`dlv`を見ていたら削除しておく
`PATH`の設定の仕方によっては、先ほどできた`dlv`がまだ参照されていないかもしれない。`which`コマンドで先ほどのディレクトリ以下の`dlv`が参照されるようになっているか確認し、別のバイナリが呼ばれているならば削除しておく。


```bash
$ which dlv
/usr/local/bin/dlv
$ rm /usr/local/bin/dlv
```


これでVScodeを再起動し、改めてデバッグを実行してみるとちゃんとデバッグが可能になった。
