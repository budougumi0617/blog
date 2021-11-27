+++
title= "[Notion] 完了ステータスのタスクの数を集計する（あるDBのSelectプロパティの特定値の数を集計する）"
date= 2021-11-26T18:30:15+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["notion"]
tags = ["notion","gtd"]
keywords = ["Notion","タスク管理","Rollup","GTD"]
twitterImage = "logos/notion.jpeg"
+++

最近Notionを使い始めた。ステータスプロパティを追加した「タスク」DBでタスク管理をしている。  
「Done」タスクの数を数えるには少し回り道が必要だったのでメモしておく。  
なお、この記事は2021/11/26時点のNotionの機能をベースに記載されている。

<!--more-->
# TL;DR
- Notionで「タスク」と「週報」のDBを作っている
    - 「タスク」には`Select`型の「ステータス」プロパティを作っている
- 「週報」ページのRollupで「`Done`ステータスのタスク数」を計算できなかった
- 「タスク」に`prop("Status") == "Done"`という`Formula`型の`Checked`プロパティを作ってこれをRollupで集計するとできた

![カスタマイズ結果](/2021/11/26_notion_result.png)


# Notionでタスク管理
最近Notionでタスク管理を始めた。
フォーマットは次の記事の「タスク」と「週報レトロスペクティブ」の作り方とデータベースを参考にカスタマイズしている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.bsoundarya.com/2021-notion-gtd-weekly-planner/" data-iframely-url="//cdn.iframe.ly/w2WCIDM?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.notion.so" data-iframely-url="//cdn.iframe.ly/fGosCWC?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

タスクにはこんな感じで`Select`プロパティとしてステータスを用意している。

![タスクのイメージ](/2021/11/26_notion_task.png)

タスクはDBのページとして登録しており、週報用のDBのページ多:多のリレーションを貼っている。  
（同じタスクを数週間使いまわしていたりするので、多:多になっている）

![週報のイメージ](/2021/11/26_notion_weekly.png)

# Selectプロパティの特定ステータスの数はRollupでカウントできない

週末に「今週どれくらいタスクを消化できたか？」を計測するため、タスクのステータスが「Done」になっているタスクの数を数えたかった。
疑似SQL的に言うとこんな感じ。
```sql
SELECT COUNT(*)
FROM `タスク'`
WHERE `タスク`.`ステータス` = 'Done'
AND `タスク`.`Week` CONTAINS '調べたい週のページ'
```

Notionには関連DBの特定要素を集計するロールアップというプロパティがあるのでこれでできるかと思った。

- リレーションとロールアップ
    - https://extns.notion.site/e730a4e9ef214cf09b9d6e8428354510

しかし、次のようにロールアップを選択しても`Select`型のプロパティの特定値をカウントすることはできないようだった。

| RELATION | All tasks|
|---|---|
| PROPERTY  | `Status` |
| CALCULATE | ??? |


2021/11/26現在のロールアップでは次のような集計しかできないので、「`Done`の数だけ数える」ができない。

- Show original
- Show unique values
- Count all
- Count values
- Count unique values
- Count empty
- Count not empty
- Percent empty
- Percent not empty

# タスク側で「`Done`かどうか」をみるFomulaを作っておく
毎回Doneの数を数えたくなかったので、隠しプロパティを設定することで解決した。
`Select`プロパティは集計できないようだったので、各タスク側に`Status`プロパティが`Done`だったらチェック状態になる`Formula`プロパティを設定した。
設定内容は次の通り。

```
prop("Status") == "Done"
```

これを使えば、各タスクに「`Status` == `Done`のときは自動でチェック状態」になるプロパティが設定できる。
このような隠しプロパティを用意して、あとは週報側でこのプロパティを集計するロールアップを設定すれば良い。

| RELATION | All tasks|
|---|---|
| PROPERTY  | `done` |
| CALCULATE | `checked` |

スクショは27タスクがこの週報に登録されていて、9タスクが`Done`ステータスになっている集計ができている状態。

![カスタマイズ結果](/2021/11/26_notion_result.png)

# 終わりに
`Done`の数は自動で集計できたが、これはリレーショナルな情報なので、未完のタスクが翌週`Done`になると数字が更新されてしまう。
そうすると年末に振り返ったとき、すべての週で登録したタスクがすべて`Done`ステータスになりかねない。
なので、週末振り返るときに集計された`Done`の数を固定値として保存するプロパティも用意してある（ここはRollupの値を手動でコピペするしかない）。


プログラマとしてはNotionはノーコードでいろいろなデータの表現をリレーショナルにできるのでおもしろい。
あまりカスタマイズしすぎるとあとでメンテできなくなりそうだが、適度にカスタマイズして利用していきたい。

# 参考
- リレーションとロールアップ
    - https://extns.notion.site/e730a4e9ef214cf09b9d6e8428354510
- Planning Your 2021 With Notion: My Weekly GTD Planner
    - https://www.bsoundarya.com/2021-notion-gtd-weekly-planner/
- 2021 GTD Planner & Retrospective
    - https://www.notion.so/2021-GTD-Planner-Retrospective-979c56adecba49fe855fea1e9b13729e

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B09JS3D9C6&linkId=1d219b07acacb2d9e8c732fb0bcaf8f4"></iframe>