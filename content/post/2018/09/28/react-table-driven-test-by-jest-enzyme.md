+++
title = "Jest( >23.0.0 )、enzymeでReactのテーブル駆動テストを行う #react #test"
date = 2018-09-28T09:21:48+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["react", "test"]
tags = ["react", "tdt", "jest", "enzyme","test"]
+++


Reactでもテーブル駆動テスト(データ駆動テスト)がしたいと思い、Jestを使ってみた。
ステートレスなコンポーネントがちゃんと設計できていれば入出力は冪等になるので、Reactとテーブル駆動テストは非常に相性がよさそう。
enzymeを使えばDOMアクセスも簡単だった。
ただ、Jestのバージョンが23.0.0以上じゃないと`each`メソッドが使えないので、`create-react-app`で作ったプロジェクトの場合は`eject`する必要があった。

JestはJavascriptで`rspec`のようなテストが書けるツール。enzymeはairbnbが作成したいい感じにDOMにアクセスできるAPIライブラリ。

- Jest
  - https://jestjs.io/
- enzyme
  - http://airbnb.io/enzyme/

<!--more-->

# TL;DR
- `create-react-app`で作ったプロジェクトの場合は`eject`してJestをアップグレードする
- `enzyme`を`yarn add`しておくとテストがラク
- `setupTest.js`を足してテストを書けば終わり
- `yarn test -watch`でホットリロードしながらテストができる
- `node --inspect-brk node_modules/.bin/jest --runInBand`を使えばChromeコンソールでいろいろ試せる

サンプルリポジトリは以下。

- tdt-react-jest-enzyme
  - https://github.com/budougumi0617/tdt-react-jest-enzyme

```react
import React from 'react';
import { shallow  } from 'enzyme';

import App from './App';

const emptyProps ={
  text: 'test'
};

describe('App', () => {
  describe('App-title', () => {
    const cases = [
      ['simple test1', { text: 'Jest'}, 'Welcome to Jest'],
      ['simple test2', { text: 'enzyme'}, 'Welcome to enzyme']
    ];
    describe.each(cases)('change Props', (title, override, expected) => {
       it(title, () => {
         const customProps = Object.assign({}, emptyProps, override);
         const wrapper = shallow(<App {...customProps} />);
         expect(wrapper.instance().props.text).toBe(customProps.text);
         expect(wrapper.find('.App-title').text()).toBe(expected);
       });
    });
  });
});
```

利用した各々のバージョンは次のとおり。

```
$ create-react-app --version
1.5.2

$ node -v
v10.8.0

$ yarn -v
1.9.4

$ egrep "react|jest|enzyme" package.json
"jest": "23.6.0",
"react": "^16.5.2",
"react-dom": "^16.5.2",
"enzyme": "^3.6.0",
"enzyme-adapter-react-16": "^1.5.0",
"react-test-renderer": "^16.5.2"
....
```

# プロジェクトの準備
まず、テーブル駆動テストを行うReactプロジェクトを作成する。

```
$ create-react-app tdt-react-jest-enzyme
```


テーブル駆動テストを利用しない場合は、このまま`enzyme`をインストールすればよい。
Jestについては`create-react-app`コマンドでプロジェクトを作成した時点でインストールされている。
が、`create-react-app`の最新版（1.5.2）でも、プリインストールされているJestのバージョンは20.X系だった（`eject`するとわかる）。


```
"jest": "20.0.4",
```

Jestでテーブル駆動をするために必要な`describe.each`は23.X系からなので利用できない。

- `describe.each(table)(name, fn, timeout)`
  - https://jestjs.io/docs/en/api#describeeachtable-name-fn-timeout

- Jest Release Notes
  - https://github.com/facebook/jest/blob/master/CHANGELOG.md#2300

そのため、`yarn eject`で`create-react-app`プロジェクトを解体したあと、Jestをアップグレードする。

```bash
$ yarn eject
$ yarn upgrade jest --latest
$ yarn add -D enzyme enzyme-adapter-react-16 react-test-renderer
```

`enzyme`は`create-react-app`のテンプレートに書いてあるとおりにインストールすれば良い。

- Testing Components
  - https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#testing-components

```bash
$ yarn add -D enzyme enzyme-adapter-react-16 react-test-renderer
```

あとは`enzyme`を読み込む`setupTests.js`を作成する。

```javascript
src/setupTests.js
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

configure({ adapter: new Adapter() });
```

Jestが読み込むように`package.json`に以下の記述を追加する。

- Jest - setupTestFrameworkScriptFile [string]
  - https://jestjs.io/docs/en/configuration.html#setuptestframeworkscriptfile-string

```diff
...
  "jest": {
...
      "mjs"
-    ]
+    ],
+    "setupTestFrameworkScriptFile": "<rootDir>/src/setupTests.js"
  },
...
```

これでプロジェクトでテーブル駆動テストができる状態にできた。
あとは以下の命名規則に則ってテストファイルを作成して`yarn test`を実行すればテストが始まる。


- Jest - testRegex [string]
  - https://jestjs.io/docs/en/configuration#testregex-string

# Jestでテーブル駆動テストを書く

まず、テスト対象のコンポーネントを作成する。今回は`create-react-app`で出来た`src/App.js`を少し改良した。


```javascript
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Welcome to { this.props.text }</h1>
        </header>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    );
  }
}

App.propTypes = {
  text: PropTypes.string
};

App.defaultProps = {
  text: 'React'
};

export default App;
```

無修正だと本当にImmutableなため、`Props`をもたせて文字列が変わるようにしてある。
テストは以下の通り。`__tests__`ディレクトリに入れるほうが好きだが、今回は`src/App.test.js`ファイルがあるのでそれを修正した。

```javascript
import React from 'react';
import { shallow  } from 'enzyme';

import App from './App';

const emptyProps ={
  text: 'test'
};

describe('App', () => {
  describe('App-title', () => {
    const cases = [
      ['simple test1', { text: 'Jest'}, 'Welcome to Jest'],
      ['simple test2', { text: 'enzyme'}, 'Welcome to enzyme']
    ];
    describe.each(cases)('change Props', (title, override, expected) => {
       it(title, () => {
         const customProps = Object.assign({}, emptyProps, override);
         const wrapper = shallow(<App {...customProps} />);
         expect(wrapper.instance().props.text).toBe(customProps.text);
         expect(wrapper.find('.App-title').text()).toBe(expected);
       });
    });
  });
});
```

Rubyを書いているひとならば、文法が似ているのではないだろうか？
`cases`で作成したテストデータのテーブルの内容の数だけ、3回目の`describe`内が実行される。
これを`yarn test`で実行してみると、テストが2ケース実行されているのがわかる。

```bash
 PASS  src/App.test.js
  App
    App-title
      change Props
        ✓ simple test1 (5ms)
        ✓ simple test2 (1ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        0.888s, estimated 1s
Ran all test suites.
```

Jestは`yarn test --watch`コマンドで実行するとファイル更新を検知して自動でテストを再実行してくれる。
これでテーブル駆動テストをしながらテスト駆動開発をすることができる。

# テストを書くときのTips
enzymeはDOMコンポーネントにいろいろなアクセスをすることができる。メソッドも実行することができる。
APIリファレンスはサンプルコードも多いので、一通り眺めるとだいたいやりかたが載っている。
Matcherの書き方も、JestのAPIリファレンスの情報量が多いので公式を見れば良い。

- enzyme - API Reference
  - http://airbnb.io/enzyme/docs/api/
- Jest - Expect
  - https://jestjs.io/docs/en/expect


イマイチテストがうまく動かない、あるいは目的のデータにどうアクセスしたらいいのかわからないときは、
以下のコマンドでデバッガを起動してREPL的に検証できる。

- Tests are Failing and You Don't Know Why
  - https://jestjs.io/docs/en/troubleshooting#tests-are-failing-and-you-don-t-know-why

```
$ node --inspect-brk node_modules/.bin/jest --runInBand
```

# 終わりに
普段はGoを書いているので、Reactでもテーブル駆動テスト（TDT）が書きたかった。
実際、Jestは`--watch`でホットリロードテストが書けるのでTDT+TDDでかなり実装が捗っている。
また、今回は省略したが、Flowとの併用も可能だ（おそらくTypeScriptととも）。
Reduxなどに対するテストはまだ書いたことがないので、いずれやってみる。

# 参考
- golang/go Wiki - TableDrivenTests
  - https://github.com/golang/go/wiki/TableDrivenTests
- Jest
  - https://jestjs.io/
- enzyme
  - http://airbnb.io/enzyme/

