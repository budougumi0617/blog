+++
title= "LeetCodeをローカルで効率よく解くためのGo専用CLIを作った"
date= 2020-10-06T10:42:02+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","cli","leetcode"]
keywords = ["Go","LeetCode","leetgode","cli"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

LeetCodeの問題をGoを使って解いている。  
テストファーストで解くためにテストコードも自動生成するCLIを作った。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/leetgode" data-iframely-url="//cdn.iframe.ly/oK5Ai7E"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# LeetCodeとは
LeetCodeはオンライン競技プログラミングコンテストが行われるWebサービスだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://leetcode.com" data-iframely-url="//cdn.iframe.ly/UxYAT4H?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>



最低限のサービスならば無料でサービスを利用することができ、データ構造やアルゴリズムを学べる。
毎週コンテストが行われて世界中の参加者とレーティングを競ったり、過去問はいつでも挑戦できる。
過去問には実際に企業の採用面接で出されたコーディング課題もあり、コーディングテスト対策としても使われている。  
ただし、問題の中には非公式にリークされた採用問題もあるようなのでグレーなサイトという指摘もある。

# LeetCode用のCLI
LeetCodeも他の競技プログラミングコンテストサービスと同様にブラウザ上のオンラインエディタを使って解答を提出できる。  
しかし、入力補完を使ったり、テストコードを書く、あるいは`Git`を利用して解答を管理しておきたいという気持ちがある。
そのため、LeetCodeにはローカルで解答用のコードを自動生成したり、問題閲覧・提出などの基本操作を行うOSSが複数存在する。

- LeetCode - Visual Studio Marketplace
    - https://marketplace.visualstudio.com/items?itemName=LeetCode.vscode-leetcode
- https://docs.rs/leetcode-cli/0.2.25/leetcode_cli/
    - https://github.com/clearloop/leetcode-cli


例を挙げると、上記のOSSを使って次の問題を選ぶと問題に対応する関数を含んだファイルがローカルに自動生成される。

- https://leetcode.com/problems/reverse-linked-list/

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
  // 回答を実装する
}
```

OSS利用ユーザーは問題を解いたあとローカルからサーバへ解答を提出したりできる。  
しかしテストコードを作るために毎回`gotests`コマンドを実行するのがめんどくさいので`leetgode`というGo専用のコマンドを作った（上記OSSは他言語にも対応している）。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/leetgode" data-iframely-url="//cdn.iframe.ly/oK5Ai7E"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# leetgodeの使い方
`leetgode`は`go get`あるいはHomeBrewからインストールすることができる。

```bash
$ go get -u https://github.com/budougumi0617/leetgode/cmd/leetgode
# or
$ brew install budougumi0617/tap/leetgode
```

`leetgode`には次のサブコマンドを用意した。（一部まだ未実装だが、）基本的にはRustのCLIと同じものだ。

```bash
$ leetgode help
Usage: leetgode is leetcode cli for gophers.

SubCommands:
 exec     Submit solution
 generate Generate the skeleton code with the test file by id
 help     Help shows usages
 list     List problems
 pick     Pick a problem by id # TODO: ほぼデバッグコードしかでない
 test     Test solution
```

問題を解いて提出するまでの流れは次のとおりだ。
なお、事前に環境変数でトークンなどを設定しておく必要がある。

- https://github.com/budougumi0617/leetgode#requirement


## 解きたい問題の番号を探す
まず解きたい問題を探す。現状は問題番号、タイトル、難易度設定しかでないが、`list`サブコマンドで問題を一覧できる。
Webブラウザで https://leetcode.com/problemset/all/ を閲覧して番号を確認してもよい。

```bash
$ leetgode list
   1 two-sum                                                                         Easy
   2 add-two-numbers                                                                 Medium
   3 longest-substring-without-repeating-characters                                  Medium
   4 median-of-two-sorted-arrays                                                     Hard
   5 longest-palindromic-substring                                                   Medium
   6 zigzag-conversion                                                               Medium
   7 reverse-integer                                                                 Easy
   8 string-to-integer-atoi                                                          Medium
   9 palindrome-number                                                               Easy
  10 regular-expression-matching                                                     Hard
...
```
## 問題番号でコードを自動生成する
解きたい問題が見つかったら`generate`サブコマンドでファイルを生成する。

```bash
# 問題番号でコードを自動生成する
$ leetgode generate 206

# ローカルに提出用関数のファイルとテストコードが自動生成される。
$ ls
206.reverse-linked-list.go      206.reverse-linked-list_test.go
```

生成されるコードはこんな感じ。`reverseList`関数を実装して提出することになる。

```go
$ cat 206.reverse-linked-list.go
package main

/**
 * <p>Reverse a singly linked list.</p>

<p><strong>Example:</strong></p>

<pre>
<strong>Input:</strong> 1-&gt;2-&gt;3-&gt;4-&gt;5-&gt;NULL
<strong>Output:</strong> 5-&gt;4-&gt;3-&gt;2-&gt;1-&gt;NULL
</pre>

<p><b>Follow up:</b></p>

<p>A linked list can be reversed either iteratively or recursively. Could you implement both?</p>

**/
/**
 * [1,2,3,4,5]
**/
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {

}
```

余談だが、LeetCodeは関数の入出力として問題を解けるので標準入出力をパースする類の競プロより問題に集中できる。

## 問題を解く
慣れたエディタで問題を解く。

```bash
# テストコードを書く
$ vim 206.reverse-linked-list_test.go
# 問題を解く
$ vim 206.reverse-linked-list.go
```

## オンラインテストを実行してみる
LeetCodeはオンラインサーバで提出時と同じ環境（？）を使ってテストを実行できる。  
`leetgode`では`test`サブコマンドを使ってオンラインテストを実行できる。

```bash
# サーバへ送信してオンラインテストを実行してみる
$ leetgode test 206
now sending...
test id: 206
problem title: reverse-linked-list
result: Accepted
```

## サーバへ提出する
テストで実装が問題なさそうならば`exec`サブコマンドでサーバへ解答を提出する。

```bash
$ leetgode exec 206
now sending...
executed id: 206
problem title: reverse-linked-list
result: Accepted
```

# テストコードを自動生成する
一般にGo用の機能がついているエディタでは対象としたGoのコードにたいしてテストコードする機能が提供されている。  
だいたいは`gotests`コマンドのラッパーだ。  

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/cweill/gotests" data-iframely-url="//cdn.iframe.ly/Ggl99tI"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


今回作成したleetgodeコマンドでも該当のOSSを利用してテストコードを自動生成する。  
やっていることはサーバから取得した関数に対して`gotests`の内部実装を読んでいるだけだ。  
そうすると提出用コードと一緒にテストコードも自動生成できる。  
`gotests`を利用しているので、生成されるテストコードも他のエディタで生成したテストファイルと内容は同じはずだ。

```go
package main

import (
        "reflect"
        "testing"
)

func Test_reverseList(t *testing.T) {
        type args struct {
                head *ListNode
        }
        tests := []struct {
                name string
                args args
                want *ListNode
        }{
                // TODO: Add test cases.
        }
        for _, tt := range tests {
                if got := reverseList(tt.args.head); !reflect.DeepEqual(got, tt.want) {
                        t.Errorf("%q. reverseList() = %v, want %v", tt.name, got, tt.want)
                }
        }
}
```

# 作るにあたって苦労したこと
「REST API叩いてローカルファイルをごにょごにょするだけでしょ」なんて思っていたが、結構苦労した。

- LeetCodeのAPIの仕様はドキュメント化されていないので、RustのCLIをリバースエンジニアリングして解析する必要があった（Rustわからない）
- 単純なREST APIだと思ったら一部エンドポイントががGraphQLだった
- サーバ上の実行結果はポーリングする必要があった
- リクエストに認証を付けるやり方もリバースエンジニアリングなので少々ハマった

逆にHomeBrew tapなどの準備はGitHub Actionsを使ってすぐ終わった。別途メモを作っておく。  
何事も手を動かさないとわからんなーと思った。

# 終わりに
CLIツールを作ってみて、REST APIを叩くだけで終わると思っていたが、いろいろやることがあって大変学びがあった。  
…とここまで書いたがだいぶ荒削りな部分が残っていたりするのでもう少し作り込んでいく。  
またHTMLをMarkdownなりplain textに変換する必要があるのだが、あまりStarが多いGoのライブラリがないので自作しようかなという気持ちになっている（パーサが書きたいだけ）。

# 参考
- https://github.com/budougumi0617/leetgode
- https://leetcode.com/
- https://github.com/cweill/gotests