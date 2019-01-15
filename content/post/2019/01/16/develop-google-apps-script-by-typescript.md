+++
title= "clasp/ESLint/Prettierを使ってGoogle Apps ScriptをTypeScriptで実装する #gas"
date= 2019-01-15T00:57:55+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["gas", "typescript"]
tags = ["gas", "typescript", "prettier", "tslint", "clasp"]
keywords = ["Google Apps Script", "GAS", "TypeScript", "Prettier", "ESLint", "clasp"]
twitterImage = "/2019/01/tslint-gas-prettier.png"
+++


claspというローカルでGoogle Apps Script(GAS)

![VSCodeでの実装画面](/2019/01/16-vscode.png)

<!--more-->

# TL;DR
- claspを使えばGoogle Apps Script(GAS)をローカルで開発できる
  - https://github.com/google/clasp
- claspはwebpackなどを使わずにTypeScriptで実装できる
  - https://github.com/google/clasp/blob/master/docs/typescript.md
  - 静的型付けをすることで補完などを使いながら実装することができる
- 書いたTypeScriptをそのままGitHubで管理することもできる
- 今回はTSLintとPrettierの設定も行なった
  - https://palantir.github.io/tslint/
  - https://prettier.io/
  - LintやFormatterを使いながら実装することができる


今回使っているclaspのバージョンは以下の通り。

```bash
$ clasp -v
1.7.0
```

なお、私の環境はすでにグローバル環境に`tslint`がインストールされている。

```bash
$ npm install tslint typescript -g
```

# claspとは

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/google/clasp/blob/master/docs/typescript.md" data-iframely-url="//cdn.iframe.ly/bBcWGaZ"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

また、clasp自体のインストール方法については以下の記事通りやれば良い。以降の処理はすでにログイン処理が実施済みであることを前提にする。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://qiita.com/HeRo/items/4e65dcc82783b2766c03" data-iframely-url="//cdn.iframe.ly/7HBfAEP"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# TypeScriptで実装するGASプロジェクトの作成方法

以下の
```bash
$ mkdir gas-typescript
$ cd gas-typescript
$ git init .
$ hub create
$ npm init -y
$ npm i -S @google/clasp tslint
$ npm i -S @types/google-apps-script @types/node
```

`jsconfig.json`ファイルを用意しておく。
```json
{
  "compilerOptions": {
    "lib": ["esnext"]
  }
}
```

どうせなら自動フォーマットもしておきたいので、prettierも導入しておく。

```bash
$ npm i -D tslint-config-prettier
$ npm i -D tslint-plugin-prettier
```


`tslint.json`は以下の設定を用意した。Stackdriverでログを取るには`console`系の関数を使うので、`no-console`は`false`にしてある。
また、prettierと設定が競合しないようにもろもろの設定も含めてある。その他は好みの設定にすれば良いと思う。

```json
$ cat tslint.json
{
    "defaultSeverity": "error",
    "extends": [
        "tslint:recommended",
        "tslint-config-prettier"
    ],
    "jsRules": {},
    "rules": {
        "semicolon": [
            true,
            "always"
        ],
        "prettier": true,
        "no-console": [
            false
        ]
    },
    "rulesDirectory": [
        "tslint-plugin-prettier"
    ]
}
```

VSCodeを使っている場合は、以下のプラグインをインストールしておく。

- Prettier - Code formatter
  - https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode

VSCode自体の`setting.json`に以下の設定を書いておくことでファイル保存時にprettierによる自動整形が走るようになる。(Command + Shift + Pを押したあと、`open settings(JSON)`などで検索すれば`setting.json`が直接編集できる。)
```json
{
    // *.tsは自動でフォーマットする
    "[typescript]": {
        "editor.formatOnSave": true
    },
    // *.tsxは自動でフォーマットする
    "[typescriptreact]": {
        "editor.formatOnSave": true
    }
}
```

あとはDrive上に紐づくGAS Projectを作ればよい。

```bash
$ clasp create --title "TypeScript sample" --type sheets --rootDir ./src
$ clasp pull
```

これでWeb上の状態と同期されている。Webブラウザなどでプロジェクトを開いていると、`clasp pull`したときに`js`ファイルがダウンロードされるかもしれない。

`src/appscript.json`内のタイムゾーンの設定がニューヨークになっていたので、東京に修正した。
```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER"
}
```


あとはローカルでtsファイルを追加して編集する。APIについては以下からたどればよい。


<div class="iframely-embed"><div class="iframely-responsive" style="padding-bottom: 52.5%; padding-top: 120px;"><a href="https://developers.google.com/apps-script/reference/" data-iframely-url="//cdn.iframe.ly/0NnS5jv"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

今回は簡単にStackdriverへのログ出力とスプレットシートに書き込みをする関数を含む`src/Hello.ts`を作成した。

```javascript
function main() {
  // https://developers.google.com/apps-script/reference/spreadsheet/spreadsheet
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const today = Utilities.formatDate(new Date(), "JST", "yyyy/MM/dd");
  const value = "Hello clasp";

  console.log(value);
  sheet.appendRow([today, value]);
}
```

TypeScriptを使っているので関数定義などのヘルプも表示され、未定義変数などのエラーも表示されている。また、ファイル保存時に自動フォーマットも行われる。

![VSCodeでの実装画面](/2019/01/16-vscode.png)

コードを書いたらまず`clasp push`コマンドでGoogle Drive上にコードをプッシュしてみる。

```
$ clasp push
? Manifest file has been updated. Do you want to push and overwrite? Yes
└─ src/Hello.ts
└─ src/appsscript.json
Pushed 2 files.
```

pushが出来たら`clasp open`コマンドでプロジェクトを開いて内容を確認しておく。

```bash
$ clasp open
Opening script: https://script.google.com/d/${.clasp.json内のscroptIdと同じ文字列}/edit
```

自動的にトランスポイルされた`Hello.gs`ファイルがプロジェクト内に配置されている。


![Web上のgasファイル](/2019/01/16-gas-file.png)

```javascript
var exports = exports || {};
var module = module || { exports: exports };
function main() {
    // https://developers.google.com/apps-script/reference/spreadsheet/spreadsheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var today = Utilities.formatDate(new Date(), "JST", "yyyy/MM/dd");
    var value = "Hello clasp";
    console.log(value);
    sheet.appendRow([today, value]);
}
```

Web上でスクリプトの実行を行おうとすると、許可を与える必要がある旨が表示されるので、案内に則って権限を付与する。

![Web上での警告画面](/2019/01/16-warning.png)
「詳細」をクリックした後の警告画面。

スプレットシートへのアクセスを許可する。
![Web上での認可画面](/2019/01/16-auth.png)

私はプロジェクトのページから直接スプレットシートを開く方法を知らないので一度GASのプロジェクト管理コンソールを開く。

https://script.google.com/home/my

ここで、該当プロジェクトの上にマウスオーバーすると、対応するスプレットシートへのリンクが表示されるのでクリックする。

![管理コンソール画面](/2019/01/16-manage-console.png)
ちゃんとGASで作った文字列が書き込まれていた。

![実行結果](/2019/01/16-sheet-result.png)

あとはローカルのプロジェクトをGitHubにコミットしておけばよい。
権限が正しく設定されていれば問題ないとは思うのだが、私は心配性なのでscriptIdが含まれている`.clasp.json`はコミットしないようにしておいた。

```
# .gitignoreファイルの中身
node_modules/
.vscode/
.clasp.json
```

# 終わりに
今回はローカルでGASの実装をするclaspを試してみた。TypeScriptを使えるようにしておくと補完なども効くようになってサクサクかけそうだ。
また、ついでにTSLintとPrettierなども導入した。GASのオンライエディタもかなり優秀だが、手元でLinter/Formatterを書けながらサクサク書けるのはよい。

# 参考
- TypeScript | google/clasp
  - https://github.com/google/clasp/blob/master/docs/typescript.md
- Reference Overview
  - https://developers.google.com/apps-script/reference/
- GAS のGoogle謹製CLIツール clasp
  - https://qiita.com/HeRo/items/4e65dcc82783b2766c03
- clasp が Typescript をサポートした！
  - https://qiita.com/HeRo/items/f2ce057c6b1456e896ad
- Prettier - Code formatter
  - https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode

