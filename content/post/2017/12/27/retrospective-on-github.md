+++
title= "2017年振り返り(GitHub編)"
date= 2017-12-27T09:41:01+09:00
draft = false
slug = ""
categories = ["poem"]
tags = ["Retrospective"]
author = "budougumi0617"
+++

今年を振り返るにあたって、まずGitHubの活動を振り返ってみた。  
個々のリポジトリへの記載を一年の活動の振り返りに含めるのも何か違和感があったので、ひとつの記事としてまとめておく。

# コントリビュート
第三者に向けていくつかプルリクエストを作成した。  
だいたいタイポ警察しかしてないし、コードのPRはマージされていない。(タイポの修正だけとは言え`golang/dep`にマージされたのは興奮したけれど。）  
ただ、自分としてはPRちゃんと出し始めたのは今年からなので、一歩一歩進んでいく。`Go`のOSSにコミットするのが来年の目標。


## golang/dep
https://github.com/golang/dep/pull/1281

「`dep`はgerritじゃないんだ、どうやってコントリビュートするんだろ？」って`CONTRIBUTING.md`開いたら壊れていたので直した。タイポのFix。
なおその後コードではコミット出来ていない。。。

## bugsnag/bugsnag-dotnet
https://github.com/bugsnag/bugsnag-dotnet/pull/66  
公式の`BugSnag`のクライアントツールがXamarin.Macだと動かなかったので。未マージ。されないかなあ。

## oracle-japan/cndjp2
https://github.com/oracle-japan/cndjp2/pull/1  
最近クラウドの勉強会に参加しているのだが、その資料のタイポのFix

## hnakamur/vim-go-tutorial-ja
https://github.com/hnakamur/vim-go-tutorial-ja/pull/7  
https://github.com/hnakamur/vim-go-tutorial-ja/pull/12  

今年は`vim-go`覚えるぞ！と思っているのだが、英語が全然だめなので、翻訳版にお世話になって何回も読ませていただいている。見つけたタイポのFix


--- 

# My OSS

今年は初めてOSSを公開した。今までGitHubは写経置き場のようになっていたので、ちょっと使いこなし始めた感。
初めてまったく知らない人に自分のリポジトリにスターつけてもらった。超嬉しかった。まだひとつだけだけど、もっともらえるようにならないと。

## Testable

https://github.com/budougumi0617/Testable  
https://www.nuget.org/packages/budougumi0617.Testable  

`.NET`のテストサポートツール。privateなメソッドやフィールドを無理やりテストできる。  
最初は`.NET Standard1.3`ベースだったのだが、リフレクション周りでどうしても`1.6`になってしまった。CIは`AppVeyor`を使っている。`.NET Standard`なので、本当はLinuxコンテナでもビルドできるのだが、やはり`.NET`系のビルドは`AppVeyor`が楽だ。  
このOSSについてはまたどこかでちゃんと触れておきたい。  
Nugetのサイトで見ると500弱ダウンロードされたようだ（累計）  
どこかで役に立っているのがわかるのは嬉しい。

## msstore-go(未完)

https://github.com/budougumi0617/msstore-go  
未完成。

[Windows ストア サービス/申請の作成と管理](https://docs.microsoft.com/ja-jp/windows/uwp/monetize/create-and-manage-submissions-using-windows-store-services)


上記のREST APIに対するクライアントを作っている。
機能が足りないのはもちろんだが、テスト書いたり、共通化とか出来ていないので、1月中に完成させておきたい。  
コピペの設定だがCircleCI2.0を使ってみた。

deeetさんが「[API Clientは最初に書くGolang パッケージとしても良いと思う．](http://deeeet.com/writing/2016/11/01/go-api-client/)」と言っていたが、たしかに勉強になる。

--- 

# Dotfile
なんだかんだちょいちょいいじっている。今年は当然Ruby周りの環境設定とかが増えた。

https://github.com/budougumi0617/dotfiles

大きめにいじったところと言えば`neobundle`から`dein.vim`に乗り換えた。また、上記で触れたとおり、`vim-go`を本格的に使い始めた。  
https://github.com/budougumi0617/dotfiles/blob/master/home/.vimrc  

`zsh`も`oh-my-zsh`でよしなにだったところを`zplug`に乗り換えてそれなりに取捨選択してみた。  
https://github.com/budougumi0617/dotfiles/blob/master/home/.zshrc

--- 

# Blog
https://github.com/budougumi0617/budougumi0617.github.io  
https://github.com/budougumi0617/blog  

今年はブログをちゃんと書き始めた（このブログ）

https://budougumi0617.github.io/

`Hugo`の作業用リポジトリで生成したデータを公開用の`github.io`にプッシュしている。  
https://github.com/budougumi0617/blog/blob/master/deploy.sh

ブログ書くとコミットも増えるのがやる気になる()。ブログについては別の振り返りでも触れるが来年はもっと更新する。

--- 

# 写経

## RubyTraining

https://github.com/budougumi0617/RubyTraining

以下の本の写経。（`koans`と楽しいRubyの練習問題も入っている。）

[実践Ruby on Rails 4 現場のプロから学ぶ本格Webプログラミング](http://amazon.jp/dp/B00LBPDNSY).  
[実践Ruby on Rails 4 機能拡張編](http://amazon.jp/dp/B00MWK10CS).

転職して`Ruby On Rails`もするようになったので。実務で触れる前にルーターやバリデーションについて知識を得られたのは良かった。`binding.pry`使ったデバッグなども練習できた。`docker-compose`とかも構築できた。

https://github.com/budougumi0617/RubyTraining/blob/master/baukis/docker-compose.yml

## gwp

https://github.com/budougumi0617/gwp

[Goプログラミング実践入門](http://amazon.jp/dp/4295000965)

GoのWebプログラミングを押さえておきたかったので。練習問題がなかったり、ちょっと古いので`context`は使っていなかったりで途中で挫折してしまった。


## gopl
https://github.com/budougumi0617/gopl

ここ数年の立ち向かっているgoplの練習問題。

[プログラミング言語Go](http://amazon.jp/dp/4621300253)

前職在職中の勉強会で終わらなかった練習問題を少しずつまだ解いている。あと５問くらい。今年こそ完結させるつもりだったがだめだった…  
途切れ途切れでやっているし、大半は1.6ベースで書いているので、全部終わったら改めて1からやり直さないといけない気がする。  
プログラミング言語Goという本は、ツールとかWebプログラミングしようと読み返すたびに当時あまり理解できなかったところ（忘れてしまったところ）に改めて気付かされる。  

## ElixirTraining
https://github.com/budougumi0617/ElixirTraining

日曜にやっている[AOSN勉強会](https://aosn.ws/)の教材。自分にとって初めての関数型言語で学びが大きい。  
新しい言語を覚えるときにREPLがあるのは試し試しが出来て捗る。  
一通り練習問題を解いていて、もう少しで終わる。これが終わったあとの勉強会の次の題材は選定中。自分としては`Haskell`と`Machine Learning With Go`を挙げているのでどちらかになると嬉しいなあ

## gsp

https://github.com/budougumi0617/gsp

この本の写経。

[Goならわかるシステムプログラミング](https://amazon.jp/dp/4908686033)

Goのリハビリと`vim-go`の運指を慣らすのも兼ねて。練習問題が前半の章にしかなかったので、写経しつつメモ足すって感じで進めている。継続中。

--- 

# Activity
2017年のコミット履歴。

https://github.com/budougumi0617/

## Publicなコミット =  1,051コミット
![Public and Private](/2017/12/glass-only-public-2017.png)

6月になってから毎日コミットするぞ！と決めて、一応毎日コミットできた。一部の日は旅先でスマホからREADME更新しただけとかもあるけど。。。  
とは言え大半は写経のコミット。「毎日コミットしているって何作ってるんすか？」って聞かれてちょっと恥ずかしくなったのを覚えている。
なのでもう少し本当の意味での「プログラミング」のコミットをしないと。


## Public + Privateなコミット =  1,801コミット
![Public only](/2017/12/glass-all-2017.png)

業務中のコミット量が全然少ないので来年はもっとコード書かないと、組織に所属するエンジニアとして非常にまずい。


--- 

# まとめ(KPT)
## Keep

- 継続的にコミットできた
- コントリビュートも何回かできた
- 他人に使ってもらえる自作OSSが公開できた

## Problem
- コードでコントリビュート出来ていない
- 写経のコミットが大半
- 業務でももっとコード書かないと。

## Try
- もっとコードでOSSに貢献する
- 自分のOSSももっと作って公開する
- 継続的なコミットは続ける
- GoとGCPでWebアプリを作る
- Kubernetesの理解を深める

と言ったところ！Webアプリについては、そろそろちゃんとクラウド、コンテナについて勉強したいのと、やはり自分はGoが好きなので。  
来年はもっと草生やすぞ。
