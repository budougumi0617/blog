+++
title= "2021年振り返り（GitHub編）"
date= 2021-12-26T15:00:57+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["poem"]
tags = ["Retrospective", "github"]
keywords = ["振り返り", "GitHub"]
twitterImage = "twittercard.png"
+++

今年も例年通りGitHubの活動を振り返った。

<!--more-->

# 2021年のコントリビュート
今年はTiDBのissueをいくつか潰した。
どれもテストコードのリファクタリングだったけれど、今までコードでコントリビュートしたことがほとんどなかったので少しは成長した感がある。
ただし機能追加のPRは出せなかった。

- `github.com/pingcap/tidb`
    - https://github.com/pingcap/tidb/pull/28548
    - https://github.com/pingcap/tidb/pull/28695
    - https://github.com/pingcap/tidb/pull/28891
    - https://github.com/pingcap/tidb/pull/29529
    - https://github.com/pingcap/tidb/pull/28979

# OSS
今年は結構「使える」OSSを作ることができた。  
しっかり業務で毎日つかうものを作ることができた。たまに他社の方から使用報告してもらえるので嬉しい。  
「世の中のツールで解決できない業務中の不便は自分で解決する」という[@fujiwara](https://twitter.com/fujiwara)さんや[@k1Low](https://twitter.com/k1low)さんみたいなOSS活動にあこがれているのでもっと頑張りたい。  
逆に言うと`tracer`みたいなOSSをみると「自分も同じ不便感じていたので自分でしゅっとつくれないとだめだな…」という気持ちになった。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/fujiwara/tracer" data-iframely-url="//cdn.iframe.ly/qV93q0Z?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## `github.com/budougumi0617/nrseg`
[GoのアプリにNew Relic APMを導入する時とても便利なCLIを作った](/2021/01/17/release_nrseg/)
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/nrseg" data-iframely-url="//cdn.iframe.ly/HrUY585?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

New RelicのSegmentを埋め込む自動生成ツール。  
機能追加したいけれど中がぐちゃぐちゃなのでリファクタリングしないといけない。

## `github.com/budougumi0617/action-newrelic-segment-lint`
[[Go] 新規追加した関数にNew RelicのSegmentを入れ忘れていたら警告するGitHub Actionsを作った](/2021/02/07/release_action_newrelic_segment_lint/)
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/marketplace/actions/action-newrelic-segment-lint" data-iframely-url="//cdn.iframe.ly/KTvYRKD"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

New RelicのSegmentが入っていなかったらPRでエラーを報告するGitHub Actions。  
これはチームのプロダクトのGoのリポジトリ全部に入っている。便利。

## `github.com/budougumi0617/cmpmock`
[gomockでモックメソッドの引数をいい感じに設定できるcmpmockを作った](/2021/04/25/reelase_cmpmock/)
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/cmpmock" data-iframely-url="//cdn.iframe.ly/TTjYmqy?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`gomock`使う時のutil。  
プロダクトでそこそこデカイ構造体を使ったテストコードを書くのに重宝している。

## `github.com/budougumi0617/action-pr-size-checker`
[テストコードなどは除外してからPRのサイズを警告するActionsを作った](/2021/05/07/reelase_action-pr-size-checker/)
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/marketplace/actions/action-pr-size-checker" data-iframely-url="//cdn.iframe.ly/aN9bpAm"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

[danger](https://github.com/danger/danger)などはテストコードも含めてPRのサイズをチェックする。  
テストコード削らないとPRをレビューしてもらえないのは本末転倒なので自作した。
これもチームのプロダクトのリポジトリのほとんど追加している。

## `github.com/budougumi0617/nrzap`
[[Go] go.uber.org/zapでNew Relic logs in contextをするためのライブラリを書き始めた](/2021/03/21/release_nrzap_for_newrelic_logs_in_context/)
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/nrzap" data-iframely-url="//cdn.iframe.ly/2Ijbpvo?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
New Relicの公式ライブラリはzapに対応していないのでラッパーを作った。  
これもチームのプロダクトのほとんどで使っている。

## `github.com/budougumi0617/list_func`
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/list_func" data-iframely-url="//cdn.iframe.ly/G9h6xiw?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

New RelicでPythonのaiohttpサーバを使うと自動でSegmentが入らないので設定を作るツール。  
私のPythonのスキルが低いのであまり作りこめていない。PythonでもASTいじるのは楽しい。

# Dotfile
https://github.com/budougumi0617/dotfiles

 - 2020年の差分
  - https://github.com/budougumi0617/dotfiles/compare/ff8d1e56844431b0760d72bac3705e94c4d9efc2...89b9c8714670c51b1843ca517c3df621f839821a

そこまでいじっていない。最近はほとんどJetBrains IDEで開発するようになってVimをあまりいじっていないのもある。  
不要なブランチをお掃除するエイリアスで`main`ブランチも特別視するようにした。  
`goto` コマンドの記事をみて使うようにした。ただリポジトリの移動は`ghq` + `fzf`でやってるので日常的にはあまりつかっていない。  
年末年始に業務PCを新しくするのでそこで少しいじるかもしれない。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://threkk.medium.com/how-to-use-bookmarks-in-bash-zsh-6b8074e40774" data-iframely-url="//cdn.iframe.ly/oV2j4cf?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# その他今年作ったリポジトリ
[created:>=2021-01-01 is:publicで絞った結果](https://github.com/budougumi0617?tab=repositories&q=created%3A%3E%3D2021-01-01+is%3Apublic&type=&language=&sort=)

## `github.com/budougumi0617/sample_tbls_actions`
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/sample_tbls_actions" data-iframely-url="//cdn.iframe.ly/1xA1AO2?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fauto_generated_er_graph_by_tbls_and_github_actions" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

ER図を自動生成するActionsのサンプルリポジトリ。詳しい説明は社のブログに書いた。  
個人のアウトプットとしては今年一番便利説ある。


# Activity
2021年のコミット履歴。

https://github.com/budougumi0617/

## Publicなコミット = 693コミット
![Public only](/2021/12/glass-only-public-2021.png)

- 2017年 1,051コミット
- 2018年 1,286コミット
- 2019年 1,357コミット
- 2020年 1,063コミット

例年に比べてほとんどOSS活動をしていなかった。技術書典で執筆していないのも関係あるだろう。  
プライベートの時間がほぼなくなってしまったので、こんなもんだろう感はあるがもう少し来年はコミット数ふやしていきたい。
前述のOSSのとおり、プロダクトで使えるものを量産したので質的には確実に良くなっていると思う。


## Public + Privateなコミット = 4,589コミット
![Public and Private](/2021/12/glass-all-2021.png)

- 2017年 1,801コミット
- 2018年 2,192コミット
- 2019年 3,095コミット
- 2020年 3,584コミット

数は増えている。来年はチームのスケールや生産性向上も考えたいが、自分のコミットも増やしていきたい。  
個人的にプログラマの信用貯金は結局コードを書いて貯めるものだと思っている。

--- 

# まとめ（KPT）
KPTを書く前に[去年のKPTのTry](/2020/12/15/retrospective-on-github-2020/)は以下だった。

## 去年のTry
- 業務で貢献してprivateコミットを増やす
- OSSに対して機能追加のPRを6個以上出す
- 素振りプロダクトでReactを触ってみる
- 素振りプロダクトでo11yまわりの技術をいろいろ触ってみる

Reactは「選択と集中」的にまったく触らなかった。o11yに関してはNew Relicに絞ってたがいくつかOSSも作ったしNRQLもよく書いたりできたと思う。
OSSに対する機能追加のPRはだせなかった。

## Keep
- 実用的なOSSを作ることができた
- GitHub starが50ほど増えた（findyの月間star数を累計）
- メジャーOSS（TiDB）のissueをいくつかcloseできた

日々の開発の不便をみつけてOSSで解決するのは今後も継続していきたい。  
サードパーティのOSSのissueやPRでいろいろやり取りしたのは貢献以上に意味があった。  
メンテナでも意外と知らないことあるんだな？みたいに距離を縮めることもできた。

## Problem
- Feature requestに対するPRをつくることができなかった
- 新しい言語に挑戦したりしなかった
- privateなコミットが減ってしまった

去年の目標に対して結果がほとんどでていなかった。
普段からよくOSSにはお世話になっているわけでもっと普段使っているOSSのコードを読んだりして身近からコミットするチャンスを探さないといけなそう。  
スキマ時間に使ったことがないツールの機能追加を頑張るというのは難しいものがあった。

## Try
- 仕事の不便を解決するOSSを2つ作る
- next.js+GraphQL+Hasuraあたりで何かつくる

家庭の事情でプライベートな時間がほとんどない&&[春までは執筆がある](https://twitter.com/budougumi0617/status/1470011071633440768)。  
夏くらいにはもう少しプライベートの学習時間を増やしていきたい。  
また、フロントエンド界隈の話題に全然ついていけていないので、習作でよいので何かつくりたいなと思う。

# 終わりに
毎年振り返っているが、今年は少しアウトプットに手応えがあった年だなと思った。
来年はもうちょっとstar集めたり、Golang Weeklyなどに取り上げられるツールがつくれたらなと思う。

しかし「プライベートな活動」==「publicなコミット」っていうのはわかりにくいものである。

# 関連
- [2017年振り返り(GitHub編)](/2017/12/27/retrospective-on-github/)
- [2018年振り返り(GitHub編)](/2018/12/15/retrospective-on-github/)
- [2019年振り返り(GitHub編)](/2019/12/14/retrospective-on-github-2019/) 
- [2020年振り返り(GitHub編)](/2020/12/15/retrospective-on-github-2020/)
