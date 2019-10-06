+++
title= "[Go]imported and not usedエラー・declared and not usedエラーとの向き合いかた"
date= 2019-10-06T11:02:11+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["Go"]
tags = ["golang"]
keywords = ["Go", "imported and not used", "declared and not used"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++



[先日の登壇資料](/2019/10/05/jrits-why-go-how-is-go/)に[ブコメでコメント](https://b.hatena.ne.jp/entry/4675383655814438370/comment/murishinai)いただいていたので私の考えを述べたいと思う。  
結論から書くと、やはり「単純さや簡潔性を保つため」が動機になるのだと思う。  
なお、このブログでは敬体は使わない方針なので、常体なのはご容赦願いたい。

> そういや "imported and not used" "declared and not used" でコンパイル通らなくなるのはどういう哲学なんだろうこれ。

<!--more-->

# "imported and not used"エラー

`imported and not used`は依存していないパッケージを`import`していたときに発生するエラーだ。

次のようなコードを実行しようとすると、`./prog.go:5:2: imported and not used: "log"`というエラーが発生する。

```go
package main

import (
	"fmt"
	"log"
)

func main() {
	fmt.Println("Hello, playground")
}
```

## "imported and not used"エラーに対する向き合いかた
まず、設計論としてあるパッケージが依存するパッケージは少ないほうがよいだろう。依存先のパッケージが他のパッケージに依存していた場合、さらに依存先が増えてしまう。
また、ビルドでもそうだろう。不要なコードを`import`して、使わないコードをビルドに含めるのはビルド時間の増大と、バイナリサイズの肥大化を招くだろう。
単純性・簡潔性を考えたときに、不要なパッケージに依存するデメリットは大きい。

唯一の例外例として、「直接依存していないが、そのパッケージの`init`関数を実行する必要がある」場合がある。具体的に言うと、データベースに接続したいときの各種ドライバーパッケージなどだ。
そのような場合は`blank import`で`import`する。こうすると、コード上そのパッケージを利用していなくても`imported and not used`エラーは発生しない。

```go
import (
	"database/sql"
	_ "github.com/go-sql-driver/mysql"
)

func main() {
	db, err := sql.Open("mysql", "user:password@/dbname")
}
```

- Import declarations | The Go Programming Language Specification
    - https://golang.org/ref/spec#Import_declarations

## "imported and not used"エラーの対処方法
エディタやIDEを正しく設定していると`imported and not used`エラーにはほとんど遭遇することはない。
`goimports`コマンドという`import`文を整理するコマンドがある。インストール方法は次の通り。

```bash
$ go get golang.org/x/tools/cmd/goimports
```

これを使うと不要な`import`文は削除される。

たとえば利用していない`log`パッケージを`import`している`go`ファイルに対して`goimports`コマンドを実行すると、以下の通り。

 ```bash
 $ goimports -d three_index.go
diff -u three_index.go.orig three_index.go
--- three_index.go.orig 2019-10-06 09:42:41.000000000 +0900
+++ three_index.go      2019-10-06 09:42:41.000000000 +0900
@@ -3,7 +3,6 @@

 import (
        "fmt"
-       "log"
 )

 func main() {
 ```

`-w`オプションを使えばそのままファイルを上書きしてくれる。
ただ、`goimports`コマンドをそのまま利用している人は少ないだろう。
エディタやIDEを使っていれば、ファイル保存時に`goimports`コマンドを自動的に実行してくれるはずだ。
なので、`imported and not used`エラーを見ることは通常ほとんどないと思う。
「あとで使うから`import`しておいたのにファイル保存したときに消えちゃった！」という失敗のほうが多い。

VS Codeの場合は`go`プラグインを使って所定のオプションを有効にしていれば自動で`goimports`コマンドが実行されるだろう。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://code.visualstudio.com/docs/languages/go" data-iframely-url="//cdn.iframe.ly/Dr68YV0?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Vimで`vim-go`プラグインを使っている場合は、`go_fmt_autosave`オプションが有効になっていて、`go_fmt_command`オプションに`goimports`をしておけば、ファイル保存時に自動実行される。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/fatih/vim-go/blob/80e475965eb7e4d52e049d7055b7053c8175a78d/doc/vim-go.txt" data-iframely-url="//cdn.iframe.ly/uLTqTZb?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


# "declared and not used"エラー
`declared and not used`エラーは関数やメソッド内で利用されていない変数があった場合のエラーだ。

次のようなコードは`unused`変数が利用されていないので、`./prog.go:8:6: unused declared and not used`というエラーが発生する。

```go
package main

import (
	"fmt"
)

func main() {
	var unused int
	fmt.Println("Hello, playground")
}
```

## "declared and not used"エラーに対する向き合いかた
不必要な変数を定義しておくのは明瞭性や可読性を下げるのではないだろうか。
将来的に使うかもしれないのだろうが、今使っていないならば消しておくのが単純性を保つ秘訣だろう。
（これらの原則は関数や機能レベルの話だろうが、）`KISSの原則`や`YAGNI`の考え方が近いのだろうか？

- KISSの原則
    - https://ja.wikipedia.org/wiki/KISS%E3%81%AE%E5%8E%9F%E5%89%87
- YAGNI
    - https://ja.wikipedia.org/wiki/YAGNI

## "declared and not used"エラーの対処方法
`declared and not used`エラーは該当の変数名や行数が一緒に出力されているので、言われたとおりに消してしまえばいいと思う。

「ある関数の戻り値の中に使わない値がある」場合に`declared and not used`エラーが出る場合は`_`で受け取れば問題ない。
```go
// func StdCopy(dstout, dsterr io.Writer, src io.Reader) (written int64, err error)
// 戻り値のint64は利用しない場合
_, err = stdcopy.StdCopy(&outBuf, &errBuf, aresp.Reader)
if err != nil {
	return err
}
```

逆に`declared and not used`エラーは関数やメソッド内の未使用変数にしか発生しないエラーだ。
「不要な`package`変数などもエラーで教えてほしい」というさらにストイックな人もいるだろう。
そういう場合は、`golanci-lint`などで3rdパーティ製のlinterを利用すればよい。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golangci/awesome-go-linters" data-iframely-url="//cdn.iframe.ly/GUoaq26"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 終わりに
やはり「単純さや簡潔性を保つため」が動機になるのだと思う。  
私としてはGoに慣れていると他の言語で不要な`import`や`using`があるほうが気になってしまう…
途中参考として書いた`YAGNI`などはうろ覚えだったので個人的に復習が必要だと感じた。

# 参考
- [第138回RITS技術交流会 / Why Go? How is Go?](https://speakerdeck.com/budougumi0617/why-go-how-is-go)
- [golang.org/x/tools/cmd/goimports](https://github.com/golang/tools/tree/master/cmd/goimports)
- [Formating | Go in Visual Studio Code](https://code.visualstudio.com/docs/languages/go#_formatting)
- [fatih/vim-go](https://github.com/fatih/vim-go)
- [KISSの原則](https://ja.wikipedia.org/wiki/KISS%E3%81%AE%E5%8E%9F%E5%89%87)
- [YAGNI](https://ja.wikipedia.org/wiki/YAGNI)
- [Unused Code | golangci/awesome-go-linters](https://github.com/golangci/awesome-go-linters#unused-code)


