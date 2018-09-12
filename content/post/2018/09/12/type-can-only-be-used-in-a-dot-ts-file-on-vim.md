+++
title = "Vim + ALEでReact+Flowなファイルを開くと'type aliases' can only be used in a .ts file.と怒られる #react #eslint"
date = 2018-09-12T21:15:51+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["react", "flow","vim"]
tags = ["javascript", "react", "flow", "ale","vim"]
+++


`create-react-app`を使って、React + Flowな開発を始めようとしたら以下のLintエラーがVimから出るようになってしまった。

- `8008: 'type aliases' can only be used in a .ts file.`
- `8011: 'type arguments' can only be used in a .ts file.`

(今回は)TypeScriptは使うつもりをないので`.ts`ファイルと言われても困ってしまう。  
エラーメッセージをググってもググってもVSCodeのエラー解決しか見つからなかったのでメモ。

<!--more-->

# TL;DR
- React+Flowな開発をVim+ALEを使って行おうとした
- `.ts`ファイルでないと`type`を使ってはいけないと怒られる
- `:ALEInfo`で設定を確認、`g:ale_linters`を設定して解消した。

`create-react-app`を使ってReactの学習を始めることにした。  

- https://github.com/facebook/create-react-app

静的型付けがないと厳しい・直近触ることが多いのでFlowを使うことにした。

- https://flow.org/

自分のVimはALEによる非同期の静的解析を走らせている。

- https://github.com/w0rp/ale

ReduxのTutorialを少しいじった以下のようなサンプルコードをプロジェクト内に置いてVimで開いたところ、冒頭に記載したエラーが出てくるようになった。

```react
// @flow

import * as React from 'react';

type Props = {
  onClick: (item: {}, e: Event) => void,
  completed: boolean,
  text: string
};

export default class Todo extends React.Component<Props> {
  render() {
    return (
      <li
        onClick={this.props.onClick}
        style={{
          textDecoration: this.props.completed ? 'line-through' : 'none'
        }}
      >
        {this.props.text}
      </li>
    );
  }
}
```

# ALEの設定を確認して、tsserverの設定をlinterから外す

`.ts`ファイルじゃないとダメと言われてもTypeScriptで開発するようなプロジェクトの環境構築はしていない。  
そこでVim(ALE)の設定を確認することにした。  
Vimで`:ALEInfo`コマンドを打つと、Vim内でALEがどのような設定で動いているか確認することができる。

```
 ALEInfo
 Current Filetype: javascript
Available Linters: ['eslint', 'flow', 'flow-language-server', 'jscs', 'jshint', 'standard', 'tsserver', 'xo']
  Enabled Linters: ['eslint', 'flow', 'flow-language-server', 'jscs', 'jshint', 'standard', 'xo']
 Suggested Fixers:
  'eslint' - Apply eslint --fix to a file.
 Suggested Fixers:
  'eslint' - Apply eslint --fix to a file.
  'importjs' - automatic imports for javascript
...
```

確認したところ、TypeScript用のLinterが有効になっているようだった。

そこで、`.vimrc`ファイル内に以下の設定を書いて、`tsserver`が動かないようにした。
```vimrc
  let g:ale_linters = {
         \'javascript': ['eslint', 'flow', 'flow-language-server', 'jscs', 'jshint', 'standard', 'xo']
  \}
```

Vimを再起動して再び設定を確認したところ`tsserver`は無効になっており、Lintエラーも納まった。


```
Available Linters: ['eslint', 'flow', 'flow-language-server', 'jscs', 'jshint', 'standard', 'tsserver', 'xo']
  Enabled Linters: ['eslint', 'flow', 'flow-language-server', 'jscs', 'jshint', 'standard', 'xo']
```


# 終わりに
もろもろのReact開発環境が有識者によって設定されているプロジェクトでは何も問題なく動いていたのでVimの設定がいけないとは思っていなかった。  
まだReduxやCSSの設定をちゃんとやっていないので、また何か引っかかったらメモしておく。
