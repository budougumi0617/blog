+++
title= "gomockで順序を無視してスライスの引数を検証する"
date= 2021-12-20T10:30:49+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["test","gomock","golang"]
keywords = ["Go","Go言語","テスト","モック","ソート"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

この記事はGoアドベントカレンダー2021 その1 20日目の記事となる。  
この記事では`github.com/golang/mock`を使ったモックのメソッドで順不同なスライス引数を検証する方法を紹介する。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://qiita.com/advent-calendar/2021/go" data-iframely-url="//cdn.iframe.ly/7pSCCsM?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# TL;DR
- gomockはGoでモックを自動生成するデファクトスタンダードなツール
- モックメソッドの引数のスライスを順不同で検証したいならば`InAnyOrder`マッチャーを使えばよい
    - https://pkg.go.dev/github.com/golang/mock@v1.6.0/gomock#InAnyOrder
- 余談:順不同でスライスを検証したくなるとき
- 普段使っているライブラリのリリースと更新内容は目を通しておきましょう

https://go.dev/play/p/hxwbM2S6vrR

```go
func TestInAnyOrder(t *testing.T) {
    if match := gomock.InAnyOrder([]int{1, 2, 3}).Matches([]int{1, 3, 2}); !match {
        t.Error("want match, but not match")
    }
}
```

# gomockはGoでモックを自動生成するデファクトスタンダードなツール
`gomock`はインターフェイスに対してモックコードを自動生成してくれるツールだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/golang/mock" data-iframely-url="//cdn.iframe.ly/RivSfnz?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Goでテストコードを書くならば高確率でお世話になると思う。

# モックメソッドの引数のスライスを順不同で検証したいならば`InAnyOrder`マッチャーを使えばよい
`gomock`はモックのメソッドで受け付ける引数を検証するが、スライスが引数の場合は当然スライスの中身の順序も含めて検証してくれる。

```go
ctrl := gomock.NewController(t)
// mockgenコマンドで自動生成したモック生成関数
mobj := mock.NewMockObject(ctrl)
// []int{1, 3, 2} を受け取った場合failする
mobj.EXPECT().ReceiveIDs([]int{1, 2, 3})
```

しかし、後述するようなシーンでスライスの中身の順序を無視してテストをしたくなる。  
そんなときは`gomock#InAnyOrder`マッチャーを使ってモックの期待引数を設定すれば良い。
- https://pkg.go.dev/github.com/golang/mock@v1.6.0/gomock#InAnyOrder

```go
// []int{1, 3, 2} を受け取ってもOKになる
mobj.EXPECT().ReceiveIDs(gomock.InAnyOrder([]int{1, 2, 3}))
```

サンプルコードは次にある。
https://github.com/golang/mock/blob/0cdccf5f55d777b12c1ac5a93f607cdd1dbf5296/gomock/matchers_test.go#L148-L295

# `InAnyOrder`が必要になるとき
「スライスはデータ構造として順序も含めて等価性を検証すべきだ！」という意見はもっともだと思う。  
しかし、`map`を経由した`distinct`処理を実装した場合などで順不同なスライスが発生しうる。

## `InAnyOrder`が必要になるサンプルコード
`InAnyOrder`を使いたくなるサンプルコードを示す。実装したい機能は次のとおり

- 渡された書籍リストからDBに登録された著者のデータを返す関数
    - 単純化のため1冊の書籍は1名の著者のみ
- 書籍リストには同じ著者の書いた書籍が複数含まれうる


これをGoで実装すると次のようになる。

```go
type AuthorID int

type Book {
    AutherID AuthorID
}

type Author {
    ID AuthorID
}

// モックを作成するインターフェイス
type AuthorFinder interface {
    FindAuthors(ids []AuthorID) []*Author
}

type Sample struct {
    ag AuthorFinder
}

func (s *Sample)GetAuthors(books []*Book) []*Author {
    ids := mapDistinct(books)
    return s.ag.FindfAuthors(ids)
}

// 書籍のスライスから重複を除外した著者IDスライスを生成するmap-distinct関数
func mapDistinct(books []*Book) []AuthorID {
    d := make(map[AuthorID]struct{}, len(books))
    for _, b := range books {
        d[b.AuthorID] = struct{}{}
    }
    ids := make([]AuthorID, len(d))
    // mapは順序性を持たないデータ構造なので順不同になる
    for id := range d {
        ids = append(ids, id)
    }
    return ids
}
```

GoでMap-Distinctするストリーム処理を書こうとすると、`map`構造を挟んで重複を削除することになる。
`map`の中身は`range`ループを使って取り出せるが、`map`はデータ構造的に順序を持たないので、ここで生成されるIDが順不同になる。
（SQLの`IN`句へ渡されることになる）プロダクトコードではこの`ids`スライスはソートする必要がない。
そのため、ここで `FindAuthors(ids []AuthorID) []*Author` メソッドの`ids`引数を順不同としてモックするテストコードを書きたくなる。

# 終わりに
実は最初は「gomockではこういうモック定義ができませんが、私の自作ライブラリを使えばできます」と書くつもりだった。
しかし、ブログを書く前にgodocを確認したら`InAnyOrder`マッチャーが増えているのに気がついた[^1]。  
普段使っているパッケージのリリースと更新内容は目を通しておいたおくべきたと反省した。
`gomock`も結構前から[`gomock.Controller#Finish`メソッドを呼ぶ必要がなくなっていたり](https://pkg.go.dev/github.com/golang/mock@v1.6.0/gomock#Controller.Finish)、ずっと使っているパッケージも日々カイゼンが行われている。


[^1]: 違うネタを用意する時間がありませんでした…



ちなみに紹介しようとした自作ライブラリはこちら。  
`gomock`用の自作マッチャーなのだが、`github.com/google/go-cmp`パッケージを使っているので、`cmp.Option`インタフェースを満たすオプションを書けばモック定義でも`go-cmp`基準の検証ができる。

- [gomockでモックメソッドの引数をいい感じに設定できるcmpmockを作った](/2021/04/25/reelase_cmpmock/)

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/cmpmock" data-iframely-url="//cdn.iframe.ly/TTjYmqy?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 参考
- https://github.com/golang/mock
- https://github.com/google/go-cmp
- https://github.com/budougumi0617/cmpmock
- [gomockでモックメソッドの引数をいい感じに設定できるcmpmockを作った](/2021/04/25/reelase_cmpmock/)
