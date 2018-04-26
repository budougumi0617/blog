+++
title= "git grepでマッチしたワードを含んだ関数名を表示する/関数スコープ全体を表示する #git"
date= 2018-04-11T08:22:22+09:00
draft = false
slug = ""
categories = ["git","tips"]
tags = ["git","grep"]
author = "budougumi0617"
+++

git 2.17にすると使える(？)`-W`オプション(とついでに`-p`)が便利すぎたのでメモ。  
2018/04/11現在、Macならば`brew upgrade git`で2.1.7に更新できる。

**Git 2.17 is now available**  
https://blog.github.com/2018-04-05-git-217-released/

（？なのは https://git-scm.com/docs/git-grep/ を見ると2.15とかでも使えそうな気配がするから）

# TL;DR
- `git grep -W`で検索結果にマッチしたワードを含む関数スコープを全て表示してくれる
- `git grep -p`で検索結果にマッチしたワードを含んだ関数の名前を表示してくれる

# `git grep`

https://git-scm.com/docs/git-grep/

`git grep`サブコマンドを使うと`git`リポジトリで管理しているファイルを対象にgrepを実行してくれる。
(`git`で管理されない)`vendor`ディレクトリや`node_modules`ディレクトリは無視してくれるので便利。


# `git grep -W`で検索結果にマッチしたワードを含む関数スコープ全体を表示する
`-W`(`--function-context`)オプションをつかうとマッチしたワードが含まれる関数スコープを表示してくれる。

```
    -W, --function-context
                          show the surrounding function
```

試した感じGoとReactはいい感じで関数を表示してくれた。

普通にgrepするとマッチした行だけしかわからないが、
```
$ git grep -i hello front/src/*.js
front/src/app.js:      <h1>Hello, React-golang!</h1>
```

`-W`オプションを使うと関数全体を表示してくれる。

```bash
git grep -i -W hello front/src/*.js
front/src/app.js=class App extends React.Component {
front/src/app.js-  render() {
front/src/app.js-    return (
front/src/app.js:      <h1>Hello, React-golang!</h1>
front/src/app.js-    );
front/src/app.js-  }
front/src/app.js-}
```

Goでもこんな感じ。

```bash
$ git grep -i -W hello backend/**/*.go
backend/main.go=func handler(w http.ResponseWriter, r *http.Request) {
backend/main.go-        dump, err := httputil.DumpRequest(r, true)
backend/main.go-        if err != nil {
backend/main.go-                http.Error(w, fmt.Sprint(err), http.StatusInternalServerError)
backend/main.go-                return
backend/main.go-        }
backend/main.go-        fmt.Println(string(dump))
backend/main.go:        fmt.Fprintf(w, "<html><body>hello</body></html>\n")
backend/main.go-}
```

# `git grep -p`で検索結果にマッチしたワードを含む関数の名前を表示する
`-p`(`--show-function`)オプションを使うと関数名を表示してくれる。
IDEがあれば関数名だけわかれば十分だし、リファクタリング対象や呼び出し元(呼び出し先)を探すときなどはこちらが便利かも。

```
    -p, --show-function   show a line with the function name before matches
```

```bash
$ git grep -i -p hello front/src/*.js
front/src/app.js=class App extends React.Component {
front/src/app.js:      <h1>Hello, React-golang!</h1>
```

Goでやってもこんな感じ。
```bash
$ git grep -i -p hello backend/**/*.go
backend/main.go=func handler(w http.ResponseWriter, r *http.Request) {
backend/main.go:        fmt.Fprintf(w, "<html><body>hello</body></html>\n")
```


# 終わりに
@deeeetさんのTweeetをみて調べてみたら今回の便利なオプションを見つけてしまった。


<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="en" dir="ltr">Grepping with function context looks nice! / “Git 2.17 is now available | The GitHub Blog” <a href="https://t.co/OfE1RyJKNB">https://t.co/OfE1RyJKNB</a></p>&mdash; Taichi Nakashima (@deeeet) <a href="https://twitter.com/deeeet/status/982782674002571264?ref_src=twsrc%5Etfw">April 8, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

冒頭にも書いたとおり、`git grep`コマンドはgit管理のファイルのみを検索ターゲットとしてくれる。
逆に言うとそれくらいしかあまりメリットを感じていなかったので結局`egrep`コマンドを多用していた。
(`grep`でも`--exclude-dir=".git" --exclude-dir="vendor"`とかつければ変わらない。実行履歴からコマンド引き回せばタイプもしなくて済むし)
前後の行を見たくて今まで`-A`とか`-B`を微調整しながらgrepしていたので`-W`で一気に見れたり、関数名が拾えるのはすごい便利だ。
関数スコープがでかいコードが混ざると`-W`がうざったくなるので、関数を小さく作るいい癖もつきそう。

何気なく使っているツールを少し調べるだけでも生産性は上げられるなと改めて痛感。


## 発展途上なところ

とは言え、言語最適化されていないためグローバルなスコープでワードがマッチすると期待した結果とは異なる表示になる。

たとえば、`-W`オプションを使っている時にGoのファイルの関数コメントとマッチしたときはファイル冒頭の`import`文まで結果が表示された。

```bash
git grep -W TODO backend/**/*.go
backend/mysql/mysql.go=import (
backend/mysql/mysql.go- "log"
backend/mysql/mysql.go- "strconv"
backend/mysql/mysql.go- "time"
backend/mysql/mysql.go-
backend/mysql/mysql.go- "github.com/jinzhu/gorm"
backend/mysql/mysql.go- // database connector
backend/mysql/mysql.go- _ "github.com/jinzhu/gorm/dialects/mysql"
backend/mysql/mysql.go-)
backend/mysql/mysql.go-
backend/mysql/mysql.go:// Task TODO
backend/mysql/mysql.go-// http://doc.gorm.io/models.html
```
