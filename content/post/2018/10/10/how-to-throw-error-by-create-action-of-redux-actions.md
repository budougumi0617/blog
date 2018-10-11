+++
title = "redux-actionsのcreateActionでFlux Standard Actionに則ったerror=trueなActionを作る #react"
date = 2018-10-10T14:04:10+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["react", "redux"]
tags = ["redux-actions", "react", "flux-standard-action"]
+++
redux-actionsを使うと、Flux Standard Action(FSA)に則った`Action`を簡単に作れる。  
が、エラーがあったときのAtionはどう作ればよいのだろう。  
よく読めば公式に書いてあるのだが、直接的なコードはなかったのでメモ。

- createAction(s) | API Reference | redux-actions
  - https://redux-actions.js.org/api/createaction

<!--more-->

# TL;DR
- FSAはActionの型を定義している。
  - https://github.com/redux-utilities/flux-standard-action#actions
- `createAction(type, payloadCreator)`は通常自動的に`{type=FOO, payload=returnValue}`なオブジェクトを返す
- `payloadCreater`が`Error`を返したときは自動的に`{type=FOO, payload=Error, error=true}`なオブジェクトを返す
  - https://github.com/redux-utilities/redux-actions/blob/51de3891278dc03713d917d636f1508c0c80d44f/src/createAction.js#L29-L31

```react
// 通常は{type="READ_EVENTS", payload=response.data}なActionが生成される。
// 例外発生時は{type="READ_EVENTS", payload=Error, error=true}なActionが生成される。
export const readEvents = createAction("READ_EVENTS", () =>
  axios.get(`${RESOURCE_URL}`).then(response => {
    return response.data;
  })
);
```

動作する文中のサンプルコードのプロジェクトは以下になる。

- https://github.com/budougumi0617/sandbox-react/tree/067c5d78f046eb7caa62e0f3f68f8a222ef2b261

利用するredux-actionsのバージョンは2.6.1。各ライブラリのバージョンの詳細は以下のファイルを参照のこと。

- https://github.com/budougumi0617/sandbox-react/blob/067c5d78f046eb7caa62e0f3f68f8a222ef2b261/package.json


# FSA準拠のアクションを作る
Flux Standard ActionはReact + Redux(Flux)で利用するActionの形を定めている。

- Actions | Flux Standard Action
  - https://github.com/redux-utilities/flux-standard-action#actions

Flowtypeの記法で型を書くと、`Action`は以下のような型になる。

```react
export type ActionT<A: Action, P> = {|
  type: A,
  payload?: P | Error,
  error?: boolean,
  meta?: mixed
|};
```

|プロパティ名|意味|
|---|---|
|type|Actionを識別する一意な文字列
|payload|ロジックの結果、もしくは`Error`。Optional|
|error|処理が失敗した場合`true`を返す真偽値。Optional|
|meta|ロジックの結果には無関係なデータ。Optional|

# redux-actionsのcreateActionでActionを作る

Redux利用時はredux-actionsの`createAction(type, payloadCreator)`を使うと、この`Action`を簡単に作れる。  
第一引数に`type`に相当する情報、第二引数には`payload`に入れる情報を返す関数を渡す(引数が２つでない`createAction`もある)。  
`redux-promise`などと組み合わせると非同期処理も含めることができる。

- redux-actions
  - https://github.com/redux-utilities/redux-actions

- createAction(s) | API Reference | redux-actions
  - https://redux-actions.js.org/api/createaction

```react
import axios from 'axios';
import { createAction } from 'redux-actions';

export type ReadEventsAction = {
  type: typeof ReadEvents,
  payload?: EventMap | Error,
  error?: boolean
};

// 通常は{type="READ_EVENTS", payload=response.data}なActionが生成される。
// 例外発生時は{type="READ_EVENTS", payload=Error, error=true}なActionが生成される。
export const readEvents: void => ReadEventsAction = createAction("READ_EVENTS", () =>
  axios.get(`${RESOURCE_URL}`).then(response => {
    return response.data;
  })
);
```

# createActionで{type=FOO, payload=Error, error=true}なオブジェクトを返す
上記のサンプルコードのコメントに書いたが、内部でエラーが発生すれば自動的に`Action`のなかの`error`が`true`になる。  
`reducer`の中で使える`handleAction`関数も用意されているので、`error = true`のときの処理を`throw`に書けばよい。


```react
import { handleActions, type ActionType } from 'redux-actions';

const initialState: {
  events: EventMap | {}
} = { events: {} };

export default handleActions(
  {
    ["READ_EVENTS"]: {
      // Actionが正常に完了したときの処理
      next(state: { events: EventMap | {} }, action: ActionType<typeof readEvents>) {
        console.log(action);
        return { ...state, events: action.payload };
      },
      // Actionが異常終了したときの処理
      throw(state: { events: EventMap | {} }, action: ActionType<typeof readEvents>) {
        // エラー処理など
        return { ...state, events: {} };
      }
    }
  },
  initialState
);

```

# 最後に
"`redux-actions`を使えばいい感じに処理してくれる"のを期待して使ってみたが、ここまでもろもろ処理してくれると思っていなかった。  
ずっと「`createAction`は`payload`の値は操作できるけど、`error`を`true`にするにはどうするんだろう？」とハマっていた。  
ちゃんとドキュメントを読んでいなかったのが原因。とは言え、以下のように書いてあってもなんかピンと来ない気がする。

- createAction(type, payloadCreator)
  - https://redux-actions.js.org/api/createaction#createactiontype-payloadcreator

> NOTE: If payload is an instance of an Error object, payloadCreator will not be called.

`payloadCreator`を呼ばないと`payload`インスタンスはわからないと思うのだが...`createAction(type)`の説明から続いているのだろうか。

- createAction(type)
  - https://redux-actions.js.org/api/createaction#createactiontype

> If the payload is an instance of an Error object, redux-actions will automatically set action.error to true.

こっちの一文はわかりやすい。

# 参考
- Flux Standard Action(FSA)
  - https://github.com/redux-utilities/flux-standard-action#actions
- redux-actions
  - https://github.com/redux-utilities/redux-actions
- createAction(s) | API Reference | redux-actions
  - https://redux-actions.js.org/api/createaction
- createAction.js
  - https://github.com/redux-utilities/redux-actions/blob/master/src/createAction.js


