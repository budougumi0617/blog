+++
title= "[Go]go-chiで作ったREST APIの定義からMarkdownやJSONを動的に出力する"
date= 2018-03-11T19:37:16+09:00
draft = false
slug = ""
categories = ["go"]
tags = ["go-chi", "golang", "REST"]
author = "budougumi0617"
+++

https://github.com/go-chi/chi

`go-chi/chi`はGolangのWebサーバを作成するときに使うシンプルなHTTPルータ。  
Swagger定義ではないが、ルーティングの設定をMarkdownやJSONで動的に出力することができたのでそのメモ。

# TL;DR
- `go-chi/chi`でHTTPルーティングを設定した`chi.Router`を作成する。
- Routerの設定からMarkdownを生成すると、READMEなどに貼れるルーティングのMarkdownを生成できる。
  - `docgen.MarkdownRoutesDoc(router, newMarkdownOpts())`
- Routerの設定からJSON文字列を生成することもできる。
  - `docgen.JSONRoutesDoc(router)`
- Markdownの場合はリンクを作成する際のURL設定などいくつかのオプションが必要
- 正直`goa`のswagger定義の自動生成などと比べるとあまり活用できる気がしないのでおまけ程度の機能…？

今回試したHTTPのルーティングの定義は以下。  
https://github.com/budougumi0617/simple-json-api-by-chi/blob/master/router.go

実際に出力したMarkdownは以下にある。  
https://github.com/budougumi0617/simple-json-api-by-chi#docgen-result-formated-by-markdown

また、JSON文字列として出力した結果は以下。  
https://github.com/budougumi0617/simple-json-api-by-chi#docgen-result-formated-by-json


# 自動生成のやり方
## `go-chi/chi`でHTTPルーティングを設定した`chi.Router`を作成する。
HTTPルーティングの設定（`chi.Router`にハンドラなどを定義していく）に特に追加で行う設定はない。

```go
func GetTodoRouter() chi.Router {
	router := chi.NewRouter()
	// Set output for logging.
	middleware.DefaultLogger = middleware.RequestLogger(
		&middleware.DefaultLogFormatter{
			Logger: newLogger(),
		},
	)
	router.Use(middleware.Logger)
	router.HandleFunc("/*", Index) // WildCard.
	router.HandleFunc("/todos", TodoIndex)
	router.HandleFunc("/todos/{todoID}", TodoShow)
	router.Post("/todos", TodoCreate)
	return router
}
```

## RouterからMarkdownを生成する
Markdownを生成する場合はオプションを指定しておく必要がある。  
https://github.com/go-chi/docgen/blob/ac43d9a63f3b58b1e82922411d2de365d896ee72/markdown.go#L22-L37

```
type MarkdownOpts struct {
	// ProjectPath is the base Go import path of the project
	ProjectPath string

	// Intro text included at the top of the generated markdown file.
	Intro string

	// ForceRelativeLinks to be relative even if they're not on github
	ForceRelativeLinks bool

	// URLMap allows specifying a map of package import paths to their link sources
	// Used for mapping vendored dependencies to their upstream sources
	// For example:
	// map[string]string{"github.com/my/package/vendor/go-chi/chi/": "https://github.com/go-chi/chi/blob/master/"}
	URLMap map[string]string
}
```

`MarkdownOpts`を生成したあとは`docgen.MarkdownRoutesDoc(r chi.Router, opts docgen.MarkdownOpts) string`を呼ぶだけだ。
以下のようなMarkdown文字列が得られる

```markdown
# github.com/budougumi0617/simple-json-api-by-chi

Sample JSON API server by go-chi.

## Routes

<details>
<summary>`/*`</summary>

- [Logger](https://github.com/go-chi/chi/blob/master/middleware/logger.go#L30)
- **/***
	- _*_
		- [main.Index](/handler.go#L16)
...
```

実際に使うと以下のようになる。

----
# github.com/budougumi0617/simple-json-api-by-chi

Sample JSON API server by go-chi.

## Routes

<details>
<summary>`/*`</summary>

- [Logger](https://github.com/go-chi/chi/blob/master/middleware/logger.go#L30)
- **/***
	- _*_
		- [main.Index](/handler.go#L16)

</details>
<details>
<summary>`/todos`</summary>

- [Logger](https://github.com/go-chi/chi/blob/master/middleware/logger.go#L30)
- **/todos**
	- _POST_
		- [main.TodoCreate](/handler.go#L36)
	- _*_
		- [main.TodoIndex](/handler.go#L21)

</details>
<details>
<summary>`/todos/{todoID}`</summary>

- [Logger](https://github.com/go-chi/chi/blob/master/middleware/logger.go#L30)
- **/todos/{todoID}**
	- _*_
		- [main.TodoShow](/handler.go#L30)

</details>

Total # of routes: 3

----


謎技術？でMarkdownなのに折りたたみなどを生成できる。GitHubのREADMEとしてもちゃんと認識できていた。  
要素が意味する説明は生成されないので、予めコンテキストを共有している人同士じゃないと意味が無いかも。  
GitHubのREADMEなどに貼ると、各ハンドラ定義の実装へのリンクがちゃんと貼れるのは良い感じ。(`main.TodoShow`など)

# RouterからJSONを生成する
JSONの場合は`docgen.JSONRoutesDoc(r chi.Router) string`を呼ぶだけで以下のようなJSON文字列が出力される。


```json
  "router": {
    "middlewares": [
      {
        "pkg": "github.com/budougumi0617/simple-json-api-by-chi/vendor/github.com/go-chi/chi/middleware",
        "func": "Logger",
        "comment": "Logger is a mi...ware pkg.\n",
        "file": "github.com/budougumi0617/simple-json-api-by-chi/vendor/github.com/go-chi/chi/middleware/logger.go",
        "line": 30
      }
    ],
    "routes": {
      "/*": {
        "handlers": {
          "*": {
            "middlewares": [],
            "method": "*",
            "pkg": "",
            "func": "main.Index",
            "comment": "Index returns simple response.\n",
            "file": "github.com/budougumi0617/simple-json-api-by-chi/handler.go",
            "line": 16
          }
        }
      },
      "/todos": {
        ...
```

vendorディレクトリに入っているmiddlewareを使った場合、pkgがそのままvendorのPATHになってしまっている。  
（Markdownの場合はオプションでマッピングすることができる。）

# 終わりに
今回のサンプルコードは以下のリポジトリにある。

https://github.com/budougumi0617/simple-json-api-by-chi


MarkdownでRESTマッピングを生成してもメモレベルかもしれないが、自動生成したかったら`goa`なのかなーと思っていたので、ありがたい。  
ドキュメントを作るためにはまず`chi.Router`を構築しなければいけないのはご愛嬌…。  
もっとちゃんと使いたかったら自分でJSONからHTMLなどに変換するコンバータを作ったほうがよいのかな？




