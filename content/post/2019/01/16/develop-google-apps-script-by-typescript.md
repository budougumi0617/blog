+++
title= "Google Apps ScriptをTypeScriptで実装する(clasp/TSLint/Prettier) #gas #typescript"
date= 2019-01-16T00:57:55+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["gas", "typescript"]
tags = ["gas", "typescript", "prettier", "tslint", "clasp"]
keywords = ["Google Apps Script", "GAS", "TypeScript", "Prettier", "ESLint", "clasp"]
twitterImage = "/logos/tsgas.png"
+++


claspというGoogle Apps Script(GAS)をローカルで開発するためのツールがある。claspを使うと、TypeScriptを使ったGASのコーディングも標準で行える。
今回はclaspを使って以下の要求を満たしながらGASの開発を行う際の設定をまとめる。

- TypeScriptによる実装
- Gitによる構成管理
- Prettierによる自動フォーマット
- TSLintによる静的解析


![VSCodeでの実装画面](/2019/01/16-vscode.png)

<!--more-->

# TL;DR
- claspを使えばGoogle Apps Script(GAS)をローカルで開発できる
  - https://github.com/google/clasp
- claspはwebpackなどを使わずにTypeScriptで実装できる
  - https://github.com/google/clasp/blob/master/docs/typescript.md
  - 静的型付けをすることで補完などを使いながら実装することができる
- 書いたTypeScriptをそのままGitHubで管理することもできる
- TSLint、PrettierとVSCodeを組み合わせた開発環境も構築できる
  - TSLint（静的解析）
      - https://palantir.github.io/tslint/
  - Prettier（フォーマッター）
      - https://prettier.io/

サンプルコードは以下のリポジトリにある。

- budougumi0617/gas-typescript
  - https://github.com/budougumi0617/gas-typescript

今回使っている各ツールのバージョンは以下の通り。

```bash
$ node -v
v10.8.0
$ npm -v
6.2.0
$ clasp -v
1.7.0
$ tslint -v
5.12.1
$ prettier -v
1.15.3
```

なお、私の環境はすでにグローバル環境にTSLintとPrettierがインストールされている。今回TSLintとPrettierの詳細な説明は省略する。

```bash
$ npm i tslint -g
$ npm i prettier -g
```

# claspとは
claspはGASをローカルで開発するためのツールだ。`clasp push`とすればローカルの`js`ファイルを`gas`ファイルとしてGoogle Drive上のApps Scriptプロジェクトにアップロードできる。
また、`clasp pull`とすればDrive上の`gas`ファイルを`js`ファイルとしてローカルにダウンロードできる。**`1.5.0`からTypeScriptを標準でサポートした。とくにwebpackなどの設定を書かなくても、`ts`ファイルをアップロードすれば自動的に`gas`ファイルに変換してくれる。**

<div class="iframely-embed"><div class="iframely-responsive" style="height: 168px; padding-bottom: 0;"><a href="https://github.com/google/clasp/blob/master/docs/typescript.md" data-iframely-url="//cdn.iframe.ly/bBcWGaZ"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

clasp自体のインストール方法については以下の記事通りやれば良い。次節以降の処理はすでにログイン処理が実施済みであることを前提にする。

- GAS のGoogle謹製CLIツール clasp
  - https://qiita.com/HeRo/items/4e65dcc82783b2766c03

# TypeScriptで実装するGASプロジェクトの作成方法

今回はDrive上にプロジェクトを作成していない状態から、Googleスプレットシートと連携するGASプロジェクトをTypeScriptを使って書いていく。まずローカルGASのプロジェクトを作成する。
途中`hub create`コマンドでGitHubリポジトリも作成している（不要なら`hub`コマンドは省略してよい）。

```bash
$ mkdir gas-typescript
$ cd gas-typescript
$ git init .
$ hub create
$ npm init -y
$ npm i -S @google/clasp tslint
$ npm i -S @types/google-apps-script @types/node
```

claspのTypeScriptの[README](https://github.com/google/clasp/blob/master/docs/typescript.md)通り、`jsconfig.json`ファイルを用意しておく。
```json
{
  "compilerOptions": {
    "lib": ["esnext"]
  }
}
```

今回は自動フォーマットも利用するので、Prettierも導入しておく。

```bash
$ npm i -D prettier
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

VSCode自体の`setting.json`に以下の設定を書いておくことでファイル保存時にprettierによる自動整形が走るようになる。(`Command + Shift + P`を押したあと、`open settings(JSON)`などで検索すれば`setting.json`が直接編集できる。)
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

あとはclasp経由でDrive上に紐づくGASプロジェクトを作ればよい。

```bash
$ clasp create --title "TypeScript sample" --type sheets --rootDir ./src
$ clasp pull
```

これでWeb上のプロジェクトと関連付けられたローカルのGASプロジェクトが作成できた。
（`clasp create`と`clasp pull`の間にWebブラウザでプロジェクトを開くと、`clasp pull`したときに`js`ファイルがダウンロードされるかもしれない。

ここで自動生成された`src/appscript.json`内のタイムゾーンの設定がニューヨークになっていたので、東京に修正した。
```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER"
}
```


あとはローカルでtsファイルを追加して編集する。APIについては以下からたどればよい。

- Reference Overview | Google Apps Script
  - https://developers.google.com/apps-script/reference/

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

VSCode上で上記のファイルを編集していると、TypeScriptを使っているので関数定義などのヘルプも表示され、未定義変数などのエラーも表示されている。また、Prettierによってファイル保存時に自動フォーマットも行われる。

![VSCodeでの実装画面](/2019/01/16-vscode.png)

## Web上のGASプロジェクトから実行する

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

自動的にトランスパイルされた`Hello.gs`ファイルがプロジェクト内に配置されている。

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

以下画像の赤矢印が指す実行ボタンをクリックしてGASを実行してみる。

![Web上のgasファイル](/2019/01/16-gas-file.png)

Web上でスクリプトの実行を行おうとすると、初回は許可を与える必要がある旨が表示されるので、案内に則って権限を付与する。
以下は開いたモーダル内の「詳細」ボタンをクリックした後の警告画面。

![Web上での警告画面](/2019/01/16-warning.png)

スプレットシートへのアクセスを許可する。
![Web上での認可画面](/2019/01/16-auth.png)

許可を与えていくと、スクリプトが実行される。実行されない場合はもう一度プロジェクトの実行ボタンをクリックする。

## 実行結果の確認

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
今回はローカルでGASの実装をするclaspを試してみた。TypeScriptを使えるようにしておくと補完なども効くようになってサクサク書けそうだ。
また、ついでにTSLintとPrettierなども導入した。GASのオンライエディタもかなり優秀だが、手元でLinter/Formatterを書けながら書けるのはよい。

# 参考
- budougumi0617/gas-typescript
  - https://github.com/budougumi0617/gas-typescript
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

