+++
title= "[Go] testing.Mオブジェクトを引数にとる関数をつくるアイデア"
date= 2021-01-10T00:05:02+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","design"]
keywords = ["go","APIデザイン","test","設計"]
twitterImage = "logos/Go-Logo_Aqua.png"

+++

ライブラリを設計するときに考えたことのメモ。

<!--more-->

# TL;DR
- よいAPI設計・ライブラリ設計の1つの基準
    - 利用者が無知モードで使えること
- Goのテストコードには2つのオブジェクトが現れる
    - `*testing.M`
    - `*testing.T`
- `TestMain`でしか使ってほしくない関数は`*testing.M`を引数にすればよいのでは？というアイデア

```go
func SetupHelper(m *testing.M) {
    // そのpkgのテスト全体でで一度しか実行しない処理
}
```

# 共通ライブラリを作っている
業務で複数のGoのサービスを扱うようになり、loggingやhttp middlewareといった汎用性の高いコードを共通ライブラリとして抽出している。  
[go-kit](https://github.com/go-kit/kit)のようなOSSとして公開されているものに頼るのもよい。  
しかし組織特有のデータ構造だったり、組織ごとで異なる技術セットに最適化すると自作することになるのが多いのではないか。  
ここで提供するライブラリの機能にはテストで利用するユーティリティ関数も含む。

# 使いやすく、間違えないAPI・ライブラリ設計
さまざまなサービスの微妙に異なるユースケースの中で、あるいはGoの経験値が均一でない開発チームの中で誤使用なくライブラリを使ってもらうには明瞭なAPIデザインが求められる。  
とくに背景を共有しなくてもこちらが想定する使い方をしてもらえるのが望ましい。  
「APIデザインの極意」ではこのようなAPIを「無知モードでも利用できること」と書いている。

> それらのAPIが適切に作成されていれば、組み立ては簡単な作業になり、デバッグしたり、ソースコードを読んだり、パッチを適用したり、つまり、一般的に言えば、他の人が行った内容を理解することに時間を費やすことがなくなります。要するに、「無知モード」で行うことができます
> 
> Jaroslav Tulach. APIデザインの極意 Java/NetBeansアーキテクト探究ノート (Japanese Edition) (Kindle の位置No.1447-1449). Kindle 版. 

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B00LGJTXT8&linkId=f9b63405984cbd477c8b764066dd0670"></iframe>

- ドキュメントを読まずとも使い方が想像できる
- 反対に、間違った使い方ができないようになっている

これらが良いAPIデザインだと考える。これはライブラリやフレームワークを提供するときも同様だ。


# テストのユーティリティ関数を作る
今回はテストの中で使うヘルパー関数を例に考える。


Goのテストは`func TestXxx(*testing.T)`というシグネチャで定義し、`*testing.T`オブジェクトは一般にテストコードの中にしか出てこない。  
このテストコードの中で利用する関数を定義するときは`*testing.T`を引数に与える。  

```go
func helper(t *testing.T)
```

`*testing.T`オブジェクトを引数にする理由は、`*testing.T`オブジェクトが持つメソッドを利用できるというのが一番の動機だ。  
ただ副次的な効果として、「この関数はテストコードの中で利用する関数なんだな」という認識を与えてくれる。

同様の効果を期待して、`TestMain`関数の中でしか利用してほしくないテストユーティリティ関数は`*testing.M`を引数に持たせればよいのではと考えた。

# *testing.Mを使う
`TestMain`と言う名前の関数を定義しておくと、そのpkgのテストが実行されるとき最初に呼ばれる。

- https://golang.org/pkg/testing/#hdr-Main

```go
func TestMain(m *testing.M)
```

そのため、DBのセットアップのような事前処理を`TestMain`関数の中に定義することが多い。  
逆に言うとここで呼ぶ事前処理はpkg内で複数回呼ばれることを期待しない。

`TestMain`関数を定義するときは`*testing.M`オブジェクトを引数にとる必要がある。  
この`*testing.M`オブジェクトは`TestMain`関数の中でしか使われることはない。  
つまり、`*testing.M`オブジェクトを引数にとる関数を定義すれば、それは`TestMain`関数の中でしか使わない関数とシグネチャで主張することができる[^sync]。

[^sync]: 処理内容を`sync.Once`で一度しか実行できないようにしておけば更に万全かもしれない。

```go
func SetupHelper(m *testing.M)
```

`*testing.M`オブジェクトは`*testing.T`オブジェクトと違い、ヘルパー関数の中で利用できる便利なメソッドはない。  
しかし、テスト関数の中で`*testing.M`オブジェクトが現れることはないため、シグネチャからテスト関数の中で利用することを想定していないことを主張できる。

# 終わりに
昨今DX(Developer Experience)が盛んに叫ばれている。  
流行りだから意識するわけではないが、プラットフォームやライブラリを提供する立場になったときは利用者が如何にストレスなく利用できるかを意識しておきたい。


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B00LGJTXT8&linkId=f9b63405984cbd477c8b764066dd0670"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07N1XLQ7D&linkId=322ab4b0cf43c2fbe7cb7e5c742847ff"></iframe>