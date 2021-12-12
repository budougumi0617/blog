+++
title= "GitHub ActionsでER図を自動生成する話とNew Relicでデプロイ頻度をグラフにする話を寄稿した"
date= 2021-12-12T10:11:36+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["github","newrelic"]
tags = ["actions","newrelic","devops"]
keywords = ["GitHub Actions","DB","migration","New Relic","devops","デプロイ回数"]
twitterImage = "twittercard.png"
+++

会社のブログに寄稿したのでそちらには書かなかった個人的な感想と一緒にメモ。
<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fauto_generated_er_graph_by_tbls_and_github_actions" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>


<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fdevops_key_metrics_on_newrelic" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>



<!--more-->
# TL;DR
- 所属の開発者ブログに寄稿した
- tblsとGitHub Actionsを使ってDBマイグレーションを含むPRには自動更新したER図を追加する
    - https://devblog.thebase.in/entry/auto_generated_er_graph_by_tbls_and_github_actions
- New Relic OneでDevOpsのキーメトリクス デプロイ頻度をグラフ化する
    - https://devblog.thebase.in/entry/devops_key_metrics_on_newrelic


# tblsとGitHub Actionsを使ってDBマイグレーションを含むPRには自動更新したER図を追加する

<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fauto_generated_er_graph_by_tbls_and_github_actions" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

1記事目はER図を自動生成するGitHub Actionsを書いた話。  
チームの生産性カイゼンに対しては私のアウトプットの中で一番インパクトがデカかったと思う。  
OSSとGitHub Actionsにタダ乗りしているだけだが鮮度の高いER図がGitHub上で見れたり、これからレビューするマイグレーションファイルのER図がレビューでみれるのは個人的にも開発体験がかなりよくなった。  
自分が開発することになるプロジェクトでは全部に適用したい。  
がんばれたらActions Marketplaceでpublicにするかもしれない（マイグレーション実行部分を汎用化するのがめんどくさい）。


# New Relic OneでDevOpsのキーメトリクス デプロイ頻度をグラフ化する

<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdevblog.thebase.in%2Fentry%2Fdevops_key_metrics_on_newrelic" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

2記事目はNew Relicをつかったデプロイ頻度を計測する記事。  
感覚値ではなく実績で「自分たちの開発はイケてるのか？」を考えられるようにしていきたい。「イケてる」状態はチームメンバにとっても開発がしやすい環境のはずだから。
カジュアル面接などで「私達の開発文化はこうです、実際実績はこれです」みたいに説明できるようになったのもよかった。
来年度はもっと可視化と分析、カイゼンのフィードバックループをつなげていきたい。

# 終わりに
それぞれ半年くらい前から「書かなきゃ書かなきゃ…」と思っていて公開できていなかったので、アドベントカレンダーという締切を使って公開できてよかった。
やはりアウトプットに締切が必要…

# 参考
- BASE Advent Calendar 2021
    - https://devblog.thebase.in/advent-calendar-2021
- New Relic Advent Calendar 2021
    - https://qiita.com/advent-calendar/2021/newrelic
- https://github.com/k1LoW/tbls
- Introduction to the Event API | New Relic
    - https://docs.newrelic.com/docs/data-apis/ingest-apis/introduction-event-api