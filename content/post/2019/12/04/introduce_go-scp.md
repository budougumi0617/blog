+++
title= "OWASP/Go-SCPを読んでセキュアプログラミングとGoを学ぶ"
date= 2019-12-04T05:36:09+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["golang"]
keywords = ["Go", "Go言語", "golang", "OWASP", "セキュアコーディング", "セキュアプログラミング"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

この記事は[Go Advent Calendar 2019](https://qiita.com/advent-calendar/2019/go)の4日目の記事になる。  
3日目は[@ikawaha](https://twitter.com/ikawaha)さんの「[Goa v3 のテストをシュッとする](https://qiita.com/ikawaha/items/e0c2b3ed0fedb12f4847)]」だった。

本記事ではOpen Web Application Security Project（OWASP)が公開している`Go-SCP`リポジトリを紹介する。  
Webアプリケーションには[クロスサイトスクリプティング（XSS）][xss_wiki]や[クロスサイトリクエストフォージェリ（CSRF）][csrf_wiki]など、様々な脆弱性が潜む可能性がある。
脆弱性対策の書籍としては、[体系的に学ぶ 安全なWebアプリケーションの作り方（徳丸本）](https://www.amazon.co.jp/dp/B07DVY4H3M)などが有名だろう。  
`Go-SCP`リポジトリにはWebアプリケーションを実装する際に必要な脆弱性の知識と、Goを使った脆弱性対策の実装方法が含まれている。

- https://github.com/OWASP/Go-SCP

[xss_wiki]: https://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B5%E3%82%A4%E3%83%88%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%97%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0
[csrf_wiki]: https://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B5%E3%82%A4%E3%83%88%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%83%95%E3%82%A9%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%AA


<!--more-->



# TL;DR
- OWASPというWEBアプリケーションのセキュリティ啓発団体がある
    - https://www.owasp.org
- Goでどのようにセキュアプログラミングをすればいいのか、をOWASPがまとめたものが`Go-SCP`リポジトリ
    - https://www.owasp.org/index.php/OWASP_Go_Secure_Coding_Practices_Guide
    - https://github.com/OWASP/Go-SCP
- 電子書籍が生成されており、PDFやmobi（Kindle）で読める。GitHub上でMarkdownを読むこともできる
- Go特有で発生する問題、Webアプリケーション一般で発生する問題両方を学習することができる


<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/OWASP/Go-SCP" data-iframely-url="//cdn.iframe.ly/A9AHYEK"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

なお、`Go-SCP`リポジトリは`Creative Commons Attribution-ShareAlike 4.0 International license (CC BY-SA 4.0)`のライセンスで公開されている。
そのため、本記事で引用しているサンプルコードなども同等のライセンスになる。
また、本記事で参照している`Go-SCP`リポジトリのバージョンは2019/12/04時点最新のv2.5.6になる。

- https://creativecommons.org/licenses/by-sa/4.0/

# OWASP（Open Web Application Security Project）とは
OWASPという団体が存在するのはご存知だろうか。

- OWASP
  - https://www.owasp.org

OWASPは、Webをはじめとするソフトウェアのセキュリティ環境の現状、またセキュアなソフトウェア開発を促進する技術・プロセスに関する情報共有と普及啓発を目的としたプロフェッショナルの集まる、オープンソース・ソフトウェアコミュニティだ（[OSWAP Japan][owasp_japan] TOPページより）。
OWASPと書いて、「オワスプ」と読む。
ソフトウェア全体の脆弱性やセキュリティに関する話題を取り扱っているため、Go以外のガイドも多数作成されている。

- OWASP Backend Security Project
    - https://www.owasp.org/index.php/Category:OWASP_Backend_Security_Project
- OWASP .NET Project
    - https://www.owasp.org/index.php/Category:OWASP_.NET_Project

[owasp_japan]: https://www.owasp.org/index.php/Japan

# OWASP Go Secure Coding Practices Guide
OWASP Go Secure Coding Practices GuideはGoを使ってWeb開発を行なう際のセキュアコーディングガイドだ。

- OWASP Go Secure Coding Practices Guide
    - https://www.owasp.org/index.php/OWASP_Go_Secure_Coding_Practices_Guide

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/OWASP/Go-SCP" data-iframely-url="//cdn.iframe.ly/A9AHYEK"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

PDF、Mobi、ePub形式でも発行されている。PDF換算だと74ページになり、そこそこのボリュームがある。更新も活発で、約2年間で25回リリースされている。

- PDF、Mobi、ePubファイル一覧
    - https://github.com/OWASP/Go-SCP/tree/master/dist

内容自体は[OWASP Secure Coding Practices Quick Reference Guide][owasp_guide]の内容をカバーしているらしく、その内容をGoで書き直したガイドとなる。
対象は他言語経験済レベルの開発者を想定しているらしいが、「Tour Of Goの次に読む学習素材としても良いだろう」と書いてある。

[owasp_guide]: https://www.owasp.org/index.php/OWASP_Secure_Coding_Practices_-_Quick_Reference_Guide

# コンテンツの内容

コンテンツの内容は[src/SUMMARY.md][summary_md]に記載の通り以下の話題が網羅されている。

[summary_md]: https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/SUMMARY.md

* [Introduction](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/README.md)
* [Input Validation](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/input-validation/README.md)
    * [Validation](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/input-validation/validation.md)
    * [Sanitization](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/input-validation/sanitization.md)
* [Output Encoding](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/output-encoding/README.md)
    * [XSS - Cross-Site Scripting](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/output-encoding/cross-site-scripting.md)
    * [SQL Injection](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/output-encoding/sql-injection.md)
* [Authentication and Password Management](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/authentication-password-management/README.md)
    * [Communicating authentication data](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/authentication-password-management/communicating-authentication-data.md)
    * [Validation and Storage](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/authentication-password-management/validation-and-storage.md)
    * [Password policies](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/authentication-password-management/password-policies.md)
    * [Other guidelines](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/authentication-password-management/other-guidelines.md)
* [Session Management](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/session-management/README.md)
* [Access Control](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/access-control/README.md)
* [Cryptographic Practices](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/cryptographic-practices/README.md)
    * [Pseudo-Random Generators](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/cryptographic-practices/pseudo-random-generators.md)
* [Error Handling and Logging](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/error-handling-logging/README.md)
    * [Error Handling](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/error-handling-logging/error-handling.md)
    * [Logging](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/error-handling-logging/logging.md)
* [Data Protection](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/data-protection/README.md)
* [Communication Security](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/communication-security/README.md)
    * [HTTP/TLS](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/communication-security/http-tls.md)
    * [WebSockets](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/communication-security/websockets.md)
* [System Configuration](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/system-configuration/README.md)
* [Database Security](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/database-security/README.md)
    * [Connections](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/database-security/connections.md)
    * [Authentication](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/database-security/authentication.md)
    * [Parameterized Queries](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/database-security/parameterized-queries.md)
    * [Stored Procedures](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/database-security/stored-procedures.md)
* [File Management](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/file-management/README.md)
* [Memory Management](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/memory-management/README.md)
* General Coding Practices
    * [Cross-Site Request Forgery](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/general-coding-practices/cross-site-request-forgery.md)
    * [Regular Expressions](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/general-coding-practices/regular-expressions.md)
* [How To Contribute](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/howto-contribute.md)
* [Final Notes](https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/final-notes.md)


中身については実際に読んでいただくとして、どんな特徴があるかを簡単に紹介する。

## Goで特有に発生する問題、Webアプリケーション一般で発生する問題両方を学習することができる
実際に読んでみると、「サンプルコードがGoなだけ」ではなかった。
Goの言語仕様や標準パッケージだからこそ起きる問題にも言及してあり、たしかにGoの文法を覚えた次に読む資料として良さそうだった。

例えば、[疑似乱数生成の章][psrg]では、Go特有のSeed問題に触れられている。
ご存知の通り、GoはSeedを設定せずに[math/rand][math_rand]パッケージにある一連の乱数生成関数を使うと毎回同じ値が生成される。
```go
// OWASP/Go-SCP/src/cryptographic-practices/pseudo-random-generators.md より
package main

import "fmt"
import "math/rand"

func main() {
    fmt.Println("Random Number: ", rand.Intn(1984))
}

// 実行結果は毎回同じになる
// $ for i in {1..5}; do go run rand.go; done
// Random Number:  1825
// Random Number:  1825
// Random Number:  1825
// Random Number:  1825
// Random Number:  1825
```

この章では「`math/rand`パッケージではなく[crypto/rand][crypto_rand]パッケージを使うべき理由」がサンプルコードと一緒にまとめられている。

```go
// OWASP/Go-SCP/src/cryptographic-practices/pseudo-random-generators.md より
package main

import "fmt"
import "math/big"
import "crypto/rand"

func main() {
    rand, err := rand.Int(rand.Reader, big.NewInt(1984))
    if err != nil {
        panic(err)
    }

    fmt.Printf("Random Number: %d\n", rand)
}

// 毎回異なる結果になる
// $ for i in {1..5}; do go run rand-safe.go; done
// Random Number: 277
// Random Number: 1572
// Random Number: 1793
// Random Number: 1328
// Random Number: 1378
```

[psrg]: https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/cryptographic-practices/pseudo-random-generators.md
[math_rand]: https://golang.org/pkg/math/rand/
[crypto_rand]: https://golang.org/pkg/crypto/rand/

また、[クロスサイトスクリプティング（XSS）の章][xss]は次のようなシンプルなWebサーバのコードから始まる。

```go
package main

import "net/http"
import "io"

func handler (w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, r.URL.Query().Get("param1"))
}

func main () {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

このサンプルコードを以下の順に改修していき、セキュアな実装を学ぶ構成になっている。

1. `io`パッケージのまま悪意のあるリクエストを受け取ってみる
1. [text/template][text_tmp]パッケージを使う実装にしてみる
1. [html/template][html_tmp]パッケージを使う実装にしてみる

XSS自体の危険性を知りながら、Goの標準パッケージを使ってどのようにセキュアなコードを書けるのかが学べる。

[xss]: https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/output-encoding/cross-site-scripting.md
なお、[クロスサイトリクエストフォージェリ（CSRF）][csrf]の説明では[gorilla/csrf][gorilla_csrf]パッケージが使われており、
標準パッケージだけでは対策の実装が難しい問題についてはOSSを利用したベターな解決策も提示されている。

```go
// OWASP/Go-SCP/src/general-coding-practices/cross-site-request-forgery.md より
// 日本語コメントは私が追加したものである。
package main

import (
    "net/http"

    "github.com/gorilla/csrf"
    "github.com/gorilla/mux"
)

func main() {
    r := mux.NewRouter()
    r.HandleFunc("/signup", ShowSignupForm)
    // All POST requests without a valid token will return HTTP 403 Forbidden.
    r.HandleFunc("/signup/post", SubmitSignupForm)

    // 生成した暗号キーをセットしておくと、gorilla/csrfがPOSTメソッドなどの際にいい感じに検証してくれる。
    // Add the middleware to your router by wrapping it.
    http.ListenAndServe(":8000",
        csrf.Protect([]byte("32-byte-long-auth-key"))(r))
    // PS: Don't forget to pass csrf.Secure(false) if you're developing locally
    // over plain HTTP (just don't leave it on in production).
}

func ShowSignupForm(w http.ResponseWriter, r *http.Request) {
    // Formの中にトークンを埋め込んでおく。
    // signup_form.tmpl just needs a {{ .csrfField }} template tag for
    // csrf.TemplateField to inject the CSRF token into. Easy!
    t.ExecuteTemplate(w, "signup_form.tmpl", map[string]interface{}{
        csrf.TemplateTag: csrf.TemplateField(r),
    })
    // We could also retrieve the token directly from csrf.Token(r) and
    // set it in the request header - w.Header.Set("X-CSRF-Token", token)
    // This is useful if you're sending JSON to clients or a front-end JavaScript
    // framework.
}

func SubmitSignupForm(w http.ResponseWriter, r *http.Request) {
    // We can trust that requests making it this far have satisfied
    // our CSRF protection requirements.
}
```


これら以外の章も英語が得意ならばそれぞれ10分もかからず読める量なので、スキマ時間に少しずつ読むこともできるだろう
（私は英語が苦手なので、得意ならばもっと早く読めるかもしれない）。

[text_tmp]: https://golang.org/pkg/text/template/
[html_tmp]: https://golang.org/pkg/html/template/
[gorilla_csrf]: https://github.com/gorilla/csrf
[csrf]: https://github.com/OWASP/Go-SCP/blob/v2.5.6/src/general-coding-practices/cross-site-request-forgery.md


# 終わりに
簡単ではあるが、セキュアプログラミングの方法が記載されている`Go-SCP`の紹介をした。  
ウェブアプリケーションフレームワーク（WAF）を使っていれば、気づかずに避けられる問題も多いかもしれない。
が、Goの場合はWAFに頼らずシンプルな実装をしているところも多いだろうし、「XXXという処理にはこういう脆弱性が潜む可能性がある」と知っておくのも大事だろう。  
`Go-SCP`リポジトリは英語だが文量もそこまで多くないので、Go・セキュアプログラミング・英語の勉強にぜひ一度目を通してみると良いだろう。

[Go Advent Calendar 2019](https://qiita.com/advent-calendar/2019/go) 5日目は[ちまきん](https://twitter.com/__timakin__)さんが担当される。お楽しみに！
（今年はGoだけで毎日7記事読めるのですごい…）


# 参考
- https://www.owasp.org/index.php/OWASP_Go_Secure_Coding_Practices_Guide
- https://github.com/OWASP/Go-SCP
- [Go Advent Calendar 2019](https://qiita.com/advent-calendar/2019/go)
- [体系的に学ぶ 安全なWebアプリケーションの作り方 第2版［固定版］　脆弱性が生まれる原理と対策の実践](https://www.amazon.co.jp/dp/B07DVY4H3M)
