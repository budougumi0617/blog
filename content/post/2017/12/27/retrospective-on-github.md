+++
title= "2017年振り返り(GitHub編)"
date= 2017-12-27T09:41:01+09:00
draft = false
slug = ""
categories = ["poem"]
tags = ["Retrospective"]
author = "budougumi0617"
+++

今年のGitHubの活動を振り返る。

# コントリビュート
改めて見るとタイポ警察しかしてないし、コードのPRはマージされていない。  
ただ、自分としてはPRちゃんと出し始めたのは今年からなので、一歩一歩進んでいく。  
`Go`に対してコミットできるくらいのスキルを身につけるのが来年の目標。


## golang/dep
https://github.com/golang/dep/pull/1281

「`dep`はgerritじゃないんだ、どうやってコントリビュートするんだろ？」って`CONTRIBUTING.md`開いたら壊れていたので直した。タイポのFix。
なおその後コードではコミット出来ていない。。。

## bugsnag/bugsnag-dotnet
https://github.com/bugsnag/bugsnag-dotnet/pull/66  
公式の`BugSnag`のクライアントツールがXamarin.Macだと動かなかったので。未マージ。されないかなあ
## oracle-japan/cndjp2
https://github.com/oracle-japan/cndjp2/pull/1  
最近Kubernetesの勉強会に参加しているのだが、その資料のタイポのFix

## hnakamur/vim-go-tutorial-ja
https://github.com/hnakamur/vim-go-tutorial-ja/pull/7  
https://github.com/hnakamur/vim-go-tutorial-ja/pull/12  

今年は`vim-go`覚えるぞ！と思っているのだが、英語が全然だめなので、翻訳版にお世話になって何回も読ませていただいている。見つけたタイポのFix


--- 

# My OSS

今年は初めてOSSを公開した。

## Testable

https://github.com/budougumi0617/Testable  
https://www.nuget.org/packages/budougumi0617.Testable  

`.NET`のテストサポートツール。privateなメソッドやフィールドを無理やりテストできる。  
このOSSについてはまたどこかでちゃんと触れておきたい。  
Nugetのサイトで見ると500弱ダウンロードされたようだ（累計）  
どこかで役に立っているのがわかるのは嬉しい。

## msstore-go(未完)

https://github.com/budougumi0617/msstore-go  
未完成。

[Windows ストア サービス/申請の作成と管理](https://docs.microsoft.com/ja-jp/windows/uwp/monetize/create-and-manage-submissions-using-windows-store-services)


上記のREST APIに対するクライアントを作っている。
機能が足りないのはもちろんだが、テスト書いたり、共通化とか出来ていないので、1月中に完成させておきたい。

deeetさんが「[API Clientは最初に書くGolang パッケージとしても良いと思う．](http://deeeet.com/writing/2016/11/01/go-api-client/)」と言っていたが、たしかに勉強になる。

--- 

# Dotfile

https://github.com/budougumi0617/dotfiles

大きめにいじったところと言えば`neobundle`から`dein.vim`に乗り換えた。また、上記で触れたとおり、`vim-go`を本格的に使い始めた。  
https://github.com/budougumi0617/dotfiles/blob/master/home/.vimrc  
`zsh`も`oh-my-zsh`任せだったところを`zplug`に乗り換えた。  
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

以下の本の写経。（`koans`と楽しいRubyの練習問題も入っている。

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

この本の写経。

[プログラミング言語Go](http://amazon.jp/dp/4621300253)

勉強会で終わらなかった練習問題を少しずつ解いている。あと５問くらい。  
途切れ途切れでやっているし、大半は1.6ベースで書いているので、全部終わったら改めて1からやり直したい。  
今やっとわかってきたこともあるし、何度でも読みたい良書。  

## ElixirTraining
https://github.com/budougumi0617/ElixirTraining

日曜にやっている[AOSN勉強会](https://aosn.ws/)の教材。自分にとって初めての関数型言語で学びが大きい。  
もう少しで終わる。次の題材は選定中。自分としては`Haskell`と`Machine Learning With Go`を挙げている。

## gsp

https://github.com/budougumi0617/gsp

この本の写経。

[Goならわかるシステムプログラミング](https://amazon.jp/dp/4908686033)

Goのリハビリも兼ねて。継続中。練習問題が前半の章にしかなかったので、写経しつつメモ足すって感じで進めようかな。

--- 

# Activity


Publicなコミット
![Public only](/2017/12/glass-all-2017.png)

6月になってから毎日コミットするぞ！と決めて、一応毎日コミットできた。一部の日は旅先でスマホからREADME更新しただけとかもあるけど。。。  
とは言え大半は写経なのでもう少し本当の意味での「プログラミング」のコミットをしないと。


PublicとPrivateも合わせたもの
![Public and Private](/2017/12/glass-only-public-2017.png)

業務中のコミット量が全然少ないので来年はもっとコード書かないと会社員としてもエンジニアとしての日中の過ごし方としても少し危機感を感じている。。。


--- 

# まとめ(KPT)
## Keep

- 継続的にコミットできた
- コントリビュートも何回かできた
- 他人に使ってもらえる自作OSSが公開できた

## Problem
- コードでコントリビュート出来ていない
- 写経のコミットが大半

## Try
- もっとコードでOSSに貢献する
- 自分のOSSももっと作って公開する
- 継続的なコミットは続ける
- GoとGCPでWebアプリを作る
- Kubernetesの理解を深める

と言ったところ！Webアプリについては、そろそろちゃんとクラウド、コンテナについて勉強したいのと、やはり自分はGoが好きなので。
