+++
title= "[発表資料] React+Redux 次の一歩 補足資料 #WEBエンジニア勉強会10"
date= 2018-11-09T09:00:22+09:00
draft = false
slug = ""
categories = ["presentation", "react"]
tags = ["react", "jest","storybook", "redux", "netlify", "enzyme"]
author = "budougumi0617"
+++

WEBエンジニア勉強会 #10 (東京都, 渋谷)の発表資料と資料中の参考リンク、紹介ライブラリの導入手順をまとめた。


<!--more-->

|イベント名|WEBエンジニア勉強会 #10 (東京都, 渋谷)|
|---|---|
|URL|https://web-engineer-meetup.connpass.com/event/104523/|
|会場|株式会社ミクシィ 東京都渋谷区東1-2-20 住友不動産渋谷ファーストタワー7F|
|日時|2018/11/09(金) 19:30 〜 22:00|
|ハッシュタグ| [#WEBエンジニア勉強会10](https://twitter.com/search?q=%23WEB%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%8B%E3%82%A2%E5%8B%89%E5%BC%B7%E4%BC%9A10)|


---

- React+Redux 次の一歩
  - https://speakerdeck.com/budougumi0617/the-next-step-from-react-and-redux

<script async class="speakerdeck-embed" data-id="364eedb4738e47ba9e57d4280b2a0693" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>


## リポジトリ・バージョン情報
本記事の内容は以下のリポジトリをcloneすることですぐ試すことができる。

- https://github.com/budougumi0617/wem10-react-sample

一部ずつ確認したい場合はコミットログを参照のこと。

- https://github.com/budougumi0617/wem10-react-sample/commits/master

なお、本記事は以下のバージョンの実行環境で作成されている。

```bash
$ npm version
{ npm: '5.6.0',
  ares: '1.13.0',
  cldr: '33.0',
  http_parser: '2.8.0',
  icu: '61.1',
  modules: '59',
  napi: '3',
  nghttp2: '1.32.0',
  node: '9.11.2',
  openssl: '1.0.2o',
  tz: '2018c',
  unicode: '10.0',
  uv: '1.19.2',
  v8: '6.2.414.46-node.23',
  zlib: '1.2.11' }
$ yarn -v
1.9.4
```

ベースとなるプロジェクトは`create-react-app`によって作成されている。


```bash
$ npm install -g create-react-app
$ create-react-app --version
2.1.1
```

# ESLint + Prettier
- ESLint
  - https://eslint.org/
- Prettier
  - https://prettier.io/

まず静的解析・フォーマッタの準備をする。`ESLint`は`create-react-app`の`v2.1.1` が`v5.6.0`しか認めないようなのでバージョンを固定している。
（本当は`ESLint`は`crreate-react-app`にプリインストールされているのだが、明示的にインストールしないと違うバージョンがインストールされてハマる）

```bash
$ yarn add -D eslint@5.6.0 eslint-config-react-app eslint-plugin-import  eslint-plugin-react eslint-config-airbnb eslint-plugin-jsx-a11y
$ yarn add -D prettier prettier-eslint-cli eslint-config-prettier eslint-plugin-prettier
```

次に`.eslintrc.js`という名前で設定ファイルを作成する（細かい設定内容の説明は割愛する）。

```js
module.exports = {
  parser: "babel-eslint",
  extends: [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:prettier/recommended",
    "prettier/react"
  ],
  env: {
    browser: true
  },
  plugins: ["react", "prettier", "jsx-a11y"],
  rules: {
    "prettier/prettier": [
      "error",
      {
        singleQuote: true,
        printWidth: 100
      }
    ],
    yoda: 0,
    "no-unused-vars": 1,
    "react/prefer-stateless-function": 0,
    "react/jsx-filename-extension": [1, { extensions: [".js", ".jsx"] }],
    "jsx-a11y/href-no-hash": "off",
    "jsx-a11y/anchor-is-valid": ["warn", { aspects: ["invalidHref"] }]
  },
  globals: {
    $: false,
    describe: true,
    it: true,
    test: true,
    expect: true,
    process: true
  }
};
```

また、自動生成ファイルはLintから外れるように、以下の内容で`.eslintignore`という名前の設定ファイルも追加する。
```
node_modules
**/*.min.js
src/registerServiceWorker.js
src/setupTests.js
```

これでバージョン差異などがなければ動くはずだ。例えば、こんな感じで`src/App.js`からセミコロンを一つ削除してみてからもう一度`yarn lint`コマンドを実行してみる。

```diff
diff --git a/src/App.js b/src/App.js
index 7e261ca..8850ce6 100644
--- a/src/App.js
+++ b/src/App.js
@@ -21,7 +21,7 @@ class App extends Component {
           </a>
         </header>
       </div>
-    );
+    )
   }
```

以下のようにPrettierのエラーが出ることがわかる。

```bash
yarn lint
yarn run v1.9.4
$ eslint src/**/*.js
Warning: React version not specified in eslint-plugin-react settings. See https://github.com/yannickcr/eslint-plugin-react#configuration.

/Users/budougumi0617/go/src/github.com/budougumi0617/wem10-react-sample/src/App.js
  24:6  error  Insert `;`  prettier/prettier

✖ 1 problem (1 error, 0 warnings)
  1 error and 0 warnings potentially fixable with the `--fix` option.

error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

あとは各エディタ・IDEで上記の設定を確認するように設定すればエディタ側でエラー内容を確認したり、オートフォーマットができる。
検索サービスで`<エディタの名前> eslint prettier`と検索した通りに設定すれば大丈夫だろう。以下の画像はVS Codeで結果を表示する例だ。

![view on VS Code](/2018/11/09-lint.png)

# Jest + Enzyme
- Option 1: Shallow Rendering
  - https://github.com/facebook/create-react-app/blob/88aef1100f349f1718874f8263684fac6fa77a07/docusaurus/docs/running-tests.md#option-1-shallow-rendering
- Jest
  - https://jestjs.io/
- Enzyme
  - https://github.com/airbnb/enzyme

次にテスト用のライブラリを追加する。
`create-react-app`を使っている場合、`Jest`はプリセットに含まれている。
なので、`Enzyme`に関連するライブラリをインストールするだけで実行可能だ。

```bash
$ yarn add -D enzyme enzyme-adapter-react-16 react-test-renderer
```

`src`ディレクトリに`setupTests.js`という名前で初期化ファイルを追加しておく。

```js
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

configure({ adapter: new Adapter() });
```

`Props`を変化させるテストを試してみたいので、`src/App.js`の内容を以下に書き換える。
```js
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
          <h1 className="App-title">Welcome to {this.props.text}</h1>
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

`src/__tests__`ディレクトリを作り`src/App.test.js`をその下に移動する。
その後、`src/__tests__/App.test.js`の内容を以下に書き換える。

```js
import React from 'react';
import { shallow } from 'enzyme';

import App from './App';

const emptyProps = {
  text: 'test'
};

describe('App', () => {
  describe('App-title', () => {
    const cases = [
      ['simple test1', { text: 'Jest' }, 'Welcome to Jest'],
      ['simple test2', { text: 'enzyme' }, 'Welcome to enzyme']
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

次に`src/setupTests.js`を以下の内容で作成する。

```js
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

configure({ adapter: new Adapter() });
```

ここまで行なって`yarn test`を実行し、以下のような実行結果が得られればテストを書く準備が完了している。
Usegeの通り、`Jest`は特に何も指定しないとホットリロードモードでプロセスが終わらないので、終わるときは`q`を入力する

```bash
PASS  src/__tests__/App.test.js
  App
    App-title
      change Props
        ✓ simple test1 (13ms)
        ✓ simple test2 (2ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        5.871s
Ran all test suites related to changed files.

Watch Usage
 › Press a to run all tests.
 › Press f to run only failed tests.
 › Press p to filter by a filename regex pattern.
 › Press t to filter by a test name regex pattern.
 › Press q to quit watch mode.
 › Press Enter to trigger a test run.
```

![hot reload test](/2018/11/09-test.gif)

# スナップショットテスト
- https://jestjs.io/docs/en/snapshot-testing

`src/__tests__`ディレクトリを作り、`src/__tests__/Snapshot.test.js`を以下の内容で作成する。

```js
import React from 'react';
import App from '../App';
import renderer from 'react-test-renderer';

it('renders correctly', () => {
  const tree = renderer.create(<App text="Snapshot" />).toJSON();
  expect(tree).toMatchSnapshot();
});
```

この状態で`yarn test`を行なうと、スナップショットテストが実行され、`src/__tests__/__snapshots__/Snapshot.test.js.snap`というファイルが生成される。
ここに現在の`App`コンポーネントをレンダリングした結果が保存されて、次回以降のテストでリグレッションテストが行われる。
もし、意図的にレンダリング結果が変わる変更をした場合はこのスナップショットファイルを更新する必要がある。
そのため、以下のコマンドを`package.json`に追加しておく。

```
diff --git a/package.json b/package.json
index fc2033d..e760ea4 100644
--- a/package.json
+++ b/package.json
@@ -11,6 +11,7 @@
     "start": "react-scripts start",
     "build": "react-scripts build",
     "test": "react-scripts test",
+    "updateSnapshot": "react-scripts test --updateSnapshot",
     "eject": "react-scripts eject",
     "lint": "eslint src/**/*.js",
     "eslint-check": "eslint --print-config .eslintrc.js | eslint-config-prettier-check"
```

# Storybook
- https://storybook.js.org/

```bash
$ yarn add -D @storybook/react @storybook/addons @storybook/addon-actions @storybook/addon-links
$ yarn add -D @storybook/addon-notes @storybook/addon-knobs @storybook/addon-events
$ yarn add -D @storybook/addon-options @storybook/addon-backgrounds @storybook/addon-storyshots babel-core babel-loader
```

`package.json`にstorybook起動用のコマンドを追加する。

```diff
diff --git a/package.json b/package.json
index e760ea4..0b3918b 100644
--- a/package.json
+++ b/package.json
@@ -14,7 +14,8 @@
     "updateSnapshot": "react-scripts test --updateSnapshot",
     "eject": "react-scripts eject",
     "lint": "eslint src/**/*.js",
-    "eslint-check": "eslint --print-config .eslintrc.js | eslint-config-prettier-check"
+    "eslint-check": "eslint --print-config .eslintrc.js | eslint-config-prettier-check",
+    "storybook": "start-storybook -p 9001 -c .storybook"
   },
   "eslintConfig": {
     "extends": "react-app"
```

`.storybook`ディレクトリを作成し、`.storybook/config.js`にファイルのロード設定をする。
```js
import { configure } from "@storybook/react";
 const req = require.context("../stories", true, /.stories.js$/); // <- import all the stories at once
 function loadStories() {
  req.keys().forEach(filename => req(filename));
}
 configure(loadStories, module);
```


`.storybook/addons.js`を作成してアドオンを読み込む設定をする。

```js
import "@storybook/addon-actions/register";
import "@storybook/addon-links/register";
import "@storybook/addon-events/register";
import "@storybook/addon-notes/register";
import "@storybook/addon-options/register";
import "@storybook/addon-knobs/register";
import "@storybook/addon-backgrounds/register";
```

`stories`ディレクトリを作成し、動作確認用に`stories/index.stories.js`を作成する。

```js
import React from 'react';
import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';
import { Button } from '@storybook/react/demo';
import { withKnobs, text, boolean, number } from '@storybook/addon-knobs';
 storiesOf('Button', module)
  .addDecorator(withKnobs)
  .add('with text', () => <Button onClick={action('clicked')}>Hello Button</Button>)
  .add('with some emoji', () => (
    <Button onClick={action('clicked')}>
      <span role="img" aria-label="so cool">
        😀 😎 👍 💯
      </span>
    </Button>
  ));
 // second stories
const stories = storiesOf('Storybook Knobs', module);
 // Add the `withKnobs` decorator to add knobs support to your stories.
// You can also configure `withKnobs` as a global decorator.
stories.addDecorator(withKnobs);
 // Knobs for React props
stories.add('with a button', () => (
  <button disabled={boolean('Disabled', false)}>{text('Label', 'Hello Storybook')}</button>
));
 // Knobs as dynamic variables.
stories.add('as dynamic variables', () => {
  const name = text('Name', 'Arunoda Susiripala');
  const age = number('Age', 89);
   const content = `I am ${name} and I'm ${age} years old.`;
  return <div>{content}</div>;
});
```

`yarn storybook`を実行して`localhost:9001`にStorybookが表示されれば成功だ。
以下のようにブラウザで各コンポーネントを操作できる。

![storybook](/2018/11/09-storybook.gif)

# Netlifyでデプロイする
- https://www.netlify.com/


まずここまでのソースコードをGitHubに公開しておく。
`create-react-app`を使ったプロジェクトならばGit用の設定が済んでいるので以下のコマンドで全てコミットする。
```bash
$ git add --all
$ git commit -v -a
```

GitHubにリポジトリを作ったら、そのリポジトリのURLをリモートリポジトリとして登録し、変更をプッシュする。
```bash
$ git remote add git@github.com:${USER_NAME}/${REPO_NAME}.git
$ git pull
# rebaseしてリモートのinit commitとマージしておく
$ git rebase origin/master
$ git push origin master
```
- Netlify
  - https://www.netlify.com/

GitHub連携などで、サインアップしたあとに以下のURLをクリックする（もしくはログイン後のページで「New site from Git」ボタンをクリックする）。

- Create a new site
  - https://app.netlify.com/start

あとは作成したリポジトリを選ぶ。「3. Build options, and deploy!」ページの設定はそのまま「Deploy site」ボタンをクリックする

そのままビルドを待っていればサイトが公開される。

- Demo site
  - https://wem10-react-sample.netlify.com/

![deploy](/2018/11/09-deploy.png)

公開されるURLはランダム値になるため、上記のURLは以下のページからURLを指定の文字列に変更している。

- Site details
  - https://app.netlify.com/sites/${YOUR_REPOSITORY_NAME}/settings/general

以降はなにもしなくてもGitHubのmasterブランチにプッシュするたびに自動でビルドされて更新される。

様々な設定ができるが、例えばビルド時に通知が欲しい場合は以下に登録すればよい。自分の場合はSlackの通知などを設定している。

- Deploy notifications
  - https://app.netlify.com/sites/${YOUR_REPOSITORY_NAME}/settings/deploys#deploy-notifications

![slack notification](/2018/11/09-notify.png)

# 終わりに
「コミットログきれいなサンプルリポジトリ作っておこう！」と思って改めてプロジェクトを作成したのだが、２，３ヶ月前に`create-react-app`したときとだいぶ中身が変わっていた気がする。もし動かなかったときはバージョンを確認してもらいたい。

- https://github.com/budougumi0617/wem10-react-sample

# 参考リンク
- [Vim + ALEでReact+Flowなファイルを開くと'type aliases' can only be used in a .ts file.と怒られる #react #eslint)](/2018/09/12/type-can-only-be-used-in-a-dot-ts-file-on-vim/)
- [Jest( >23.0.0 )、enzymeでReactのテーブル駆動テストを行う #react #test](/2018/09/28/react-table-driven-test-by-jest-enzyme/)
