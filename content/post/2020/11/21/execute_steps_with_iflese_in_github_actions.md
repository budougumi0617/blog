+++
title= "ifelseやswitch文のようにGitHub Actionsのステップの実行を制御したい"
date= 2020-11-21T13:15:23+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["github","actions"]
tags = ["github","actions","git","ci"]
keywords = ["GitHub Actions","CI","GitHub"]
twitterImage = "logos/github.png"
+++


GitHub Actions上でこんな感じにステップの実行を制御する方法をまとめておく。

```
if condition1 {
  step1 実行
} else if condition2 {
  step2 実行
} else {
  step3 実行
}
```

<!--more-->

# TD;DR
- GitHub Actionsでstepに対して条件分岐を書きたかった
    - 条件AのときはStep Aを実行
    - 条件BのときはStep Bを実行
    - それ以外のときはStep Cを実行
    - 1回のジョブ中にA/B/Cはどれかひとつだけ実行されてほしい
- stepの実行可否は`if`構文で制御することができる
- 実行済みのstepの実行結果は`conclusion`コンテキストで取得できる
- これらを組み合わせて擬似的にifelseのような操作を実現できる

検証したActionsの定義は次の通り。
```yaml
name: ifelse pattern
on:
  push:
    branches:
      - master
jobs:
  ifelse-pattern:
    runs-on: ubuntu-latest
    steps:
      - name: Check out codes
        uses: actions/checkout@v2
      - name: foo
        id: foo
        if: contains(toJSON(github.event.commits.*.message), 'foo')
        run:  echo 'if step!'
      - name: bar
        id: bar
        if: steps.foo.conclusion == 'skipped'  && contains(toJSON(github.event.commits.*.message), 'bar')
        run:  echo 'elseif step!'
      - name: other
        if: "!(steps.foo.conclusion == 'success' || steps.bar.conclusion == 'success')"
        run:  echo 'else step!'
```
サンプルリポジトリは次のとおり。

- https://github.com/budougumi0617/actions-sandbox

利用しているGitHub Actionsのバージョンはv2となる。

# GitHub Actionsでswtich文のようにステップの実行を制御したい。
GitHub Actionsを個人開発だったり業務でも選択することが多くなってきた。  
慣れてくると少し凝ったことをやりたくなるのが人間の性である。

今回はActionsのワークフローでいくつか定義したステップを場合によって使い分けたいシーンが出てきた。  
そこで、if-elseのようにしてステップを使い分ける方法を調べてみた。  
擬似コードを書くと次のように感じで、定義したstepのどれかひとつだけを実行したい。

```
if condition1 {
  step1 実行
} else if condition2 {
  step2 実行
} else {
  step3 実行
}
```

# if構文とconclusionコンテキストを使いこなす
やりたいことを実現するには、`if`構文と`conclusion`コンテキストを利用すればよかった。

## `jobs.<job_id>.steps.if`
https://docs.github.com/ja/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepsif

`if`構文はそのままの意味で、`job`や`step`単位で実行条件を設定できる。  
`if`構文では条件式や関数、環境変数、コンテキストの情報を利用することができる。

## `steps.<step id>.conclusion`
https://docs.github.com/en/free-pro-team@latest/actions/reference/context-and-expression-syntax-for-github-actions#steps-context

コンテキストとは、ジョブ実行中に得られる情報だ。  
たとえば次のようなコンテキスト情報がある。

- トリガーとして渡されたコミットや実行者などのイベント情報
- ジョブ実行中の各ステップのアウトプット情報

この中に実行済みのステップの実行結果を取得できるコンテキストも存在する。

|Property name|Type|Description
|---|---|---|
steps.<step id>.conclusion|string|The result of a completed step after continue-on-error is applied. Possible values are success, failure, cancelled, or skipped. When a continue-on-error step fails, the outcome is failure, but the final conclusion is success.

前ステップが`skipped`されたときに別の条件Aを評価すれば`else if 条件A`的なステップ実行が可能そうだ。  
また、関連する他のステップがすべて成功しなかったときにステップを実行すれば、`else`的なステップ実行ができる。

## サンプルコード
実際に作成したActionsのサンプルコードが次になる。

```yaml
name: ifelse pattern
on:
  push:
    branches:
      - master
jobs:
  ifelse-pattern:
    runs-on: ubuntu-latest
    steps:
      - name: Check out codes
        uses: actions/checkout@v2
      - name: foo
        id: foo
        if: contains(toJSON(github.event.commits.*.message), 'foo')
        run:  echo 'if step!'
      - name: bar
        id: bar
        if: steps.foo.conclusion == 'skipped'  && contains(toJSON(github.event.commits.*.message), 'bar')
        run:  echo 'elseif step!'
      - name: other
        if: "!(steps.foo.conclusion == 'success' || steps.bar.conclusion == 'success')"
        run:  echo 'else step!'
```

`contains(toJSON(github.event.commits.*.message), 'foo')`は「プッシュされたコミットのコミットメッセージにfooが含まれているか」を評価するおまじないだ。
なので、上記のActionsは次のような制御フローを実現している。

- コミットメッセージに「foo」が含まれていたら、fooステップの`echo 'if step!'を実行する
- コミットメッセージに「bar」が含まれていたら、barステップの`echo 'elseif step!'を実行する
    - foobarと書かれていたらfooステップが一度だけ実行される
- fooもbarも含まれていないならば、otherステップの`echo else strep!`を実行する


実際に動かして挙動を確認してみる。

- コミットメッセージに「foobar」と書いてプッシュした時、fooステップだけ実行された
    - https://github.com/budougumi0617/actions-sandbox/runs/1434708015?check_suite_focus=true
- コミットメッセージに「barrrr」と書いてプッシュした時、barステップだけ実行された
    - https://github.com/budougumi0617/actions-sandbox/runs/1434710021?check_suite_focus=true
- コミットメッセージに「:truck: move README」と書いてプッシュした時、otherステップだけ実行された
    - https://github.com/budougumi0617/actions-sandbox/runs/1434920272?check_suite_focus=true

どれも期待通りの挙動になった。

# 終わりに
GitHub Actionsはあまり複雑なワークフローを組めないものの、覚えることが少なく、開始も簡単でいろいろやりやすい。  
コンテキストを通してプッシュされたコミットの情報にアクセスしやすいのも魅力のひとつだなと思った。

# 参考文献
- GitHub Actionsのワークフロー構文
    - https://docs.github.com/ja/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions
- GitHub Actions のコンテキストおよび式の構文
    - https://docs.github.com/ja/free-pro-team@latest/actions/reference/context-and-expression-syntax-for-github-actions