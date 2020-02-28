+++
title= "[冒頭公開]技術書典 応援祭にgolang.tokyo新刊「Gopherの休日2020冬」で参加します #技術書典"
date= 2020-02-28T09:37:18+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["Go", "golangtokyo", "技術書典"]
keywords = ["技術書典", "技術書典8", "golang.tokyo", "golangtokyo", "Go", "golang", "SOLIDの原則"]
twitterImage = "2020/02/28_shoten8.jpeg"
+++

技術書典8は中止になってしまいましたが、オンラインで開催される技術書典 応援祭に[golang.tokyo](https://golangtokyo.github.io/)も参加します。  
私は、今回の新刊である「Gopherの休日2020冬」に「GoにおけるSOLID原則」という内容で寄稿しました。  
また、その冒頭部分も公開します。気になる方はどうぞ次のリンク先よりサークルチェックをお願いします。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://techbookfest.org/event/tbf08/circle/5752290824683520" data-iframely-url="//cdn.iframe.ly/ZldmKm4"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<!--more-->

# golang.tokyo 技術書典 応援祭 新刊「Gopherの休日2020冬」について
今回の新刊は私以外にも6名以上の方々が執筆しており、以下のような内容になっています（敬称略）。

![Gopherの休日2020冬](/2020/02/28_cover.png)

- GoにおけるSOLIDの原則 / [@budougumi0617][budougumi0617]
- Go本体にコントリビュートする方法 / [@hajimehoshi][hajimehoshi]
- TUIツールを作ろう / [@gorilla0513][gorilla0513]
- GoとコンセンサスアルゴリズムRaftによる分散システム構築入門  / [@po3rin][po3rin]
- Goにおける初期化処理 / [@kaneshin0120][kaneshin]
- text/template の linter を作ろう / [@knsh14][knsh14]
- Goによるコンテナランタイム自作入門 / [`@_moricho_`][_moricho_]
- and more...

今回もすべての内容が技術書典 応援祭のための書き下ろしになっています。  
Go本体の解説やコントリビューション方法、分散システム、linter・TUI、コンテナランタイムの自作まで多種多様です。  
ページ数は現時点で計150ページ以上になっており、販売価格1,000円となります。 （なお、売上はすべてgolang.tokyoのノベルティ作成などに使われます）  
また、前回同様、[@tottie_designer][tottie]さんに表紙・裏表紙を作成していただきました。

[budougumi0617]: https://twitter.com/budougumi0617
[knsh14]: https://twitter.com/knsh14
[po3rin]: https://twitter.com/po3rin
[hajimehoshi]: https://twitter.com/hajimehoshi
[kaneshin]: https://twitter.com/tottie_designer
[tenntenn]: https://twitter.com/tenntenn
[tottie]: https://twitter.com/tottie_designer
[gorilla0513]: https://twitter.com/gorilla0513
[_moricho_]: https://twitter.com/_moricho_

# 寄稿内容について
私はオブジェクト指向設計の重要な原則であるSOLIDの原則を話題に「GoにおけるSOLID原則」という章を書きました。  
抽象クラス・具象クラスなどが存在しないGoにおいてどのようにSOLIDの原則の思想を生かしていくか、を（自分なりに解釈して）まとめています。

以下、試し読み代わりに冒頭を転載します（脚注などはブログ用に編集）。  
内容に興味がでた方はぜひ応援祭でご購入ください。冒頭に記載したとおり、この他にも様々なテーマで寄稿されています。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://techbookfest.org/event/tbf08/circle/5752290824683520" data-iframely-url="//cdn.iframe.ly/ZldmKm4"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

---

# GoにおけるSOLIDの原則
[`@budougumi0617`](https://twitter.com/budougumi0617)です。  
オブジェクト指向設計の原則を5つまとめた[SOLIDの原則](https://ja.wikipedia.org/wiki/SOLID)（`the SOLID principles`）という設計指針があることはみなさんご存じでしょう。  
本章ではSOLIDの原則にのっとった`Go`の実装について考えます。

## `SOLIDの原則`（`the SOLID principles`）とは
SOLIDの原則は次の用語リストに挙げた5つのソフトウェア設計の原則の頭文字をまとめたものです（アジャイルソフトウェア開発の奥義 第2版より引用）。

- `単一責任の原則`（`SRP`, `Single responsibility principle`）
    - クラスを変更する理由は1つ以上存在してはならない。
- `オープン・クローズドの原則`（`OCP`, `Open–closed principle`）
    - ソフトウェアの構成要素構成要素（クラス、モジュール、関数など）は拡張に対して開いて（オープン: Open）いて、修正に対して閉じて（クローズド: Closed）いなければならない。
- `リスコフの置換原則`（`LSP`, `Liskov substitution principle`）
    - `S`型のオブジェクト`o1`の各々に、対応する`T`型のオブジェクト`o2`が1つ存在し、`T`を使って定義されたプログラム`P`に対して`o2`の代わりに`o1`を使っても`P`の振る舞いが変わらない場合、`S`は`T`の派生型であると言える。
- インターフェイス分離の原則（`ISP`, `Interface segregation principle`）
    - クライアントに、クライアントが利用しないメソッドへの依存を強制してはならない。
- `依存関係逆転の原則`（`DIP`,  `Dependency inversion principle`）
    - 上位のモジュールは下位のモジュールに依存してはならない。どちらのモジュールも「抽象」に依存すべきである。「抽象」は実装の詳細に依存してはならない。実装の詳細が「抽象」に依存すべきである。

これらの原則は、ソフトウェアをより理解しやすく、より柔軟でメンテナンス性の高いものにする目的で考案されました。
各々の原則の原案者や発表時期は異なりますが、この5つの原則から頭文字のアルファベットを1つずつ取り`SOLIDの原則`としてまとめたのが`Robert C. Martin`氏です。

- Getting a SOLID start.
    - https://sites.google.com/site/unclebobconsultingllc/getting-a-solid-start

メーリングリストに投稿された`Robert C. Martin`氏のコメントを見ると、1995年時点でオープンクローズドの原則（`The open/closed principle`）などの命名がされていることがわかります。

- The Ten Commandments of OO Programming
    - https://groups.google.com/forum/m/#!msg/comp.object/WICPDcXAMG8/EbGa2Vt-7q0J

また、SOLIDの原則として5つの原則がまとめて掲載された日本語書籍は、「[アジャイルソフトウェア開発の奥義](https://www.amazon.co.jp/dp/4797347783)」になります。
SOLIDの原則の成り立ち自体についてもっと知りたい場合は、[`@nazonohito51`](https://twitter.com/nazonohito51)さんの「`「SOLIDの原則って何ですか？」って質問に答えたかった`」を読んでみるとよいでしょう。

- 「SOLIDの原則って何ですか？」って質問に答えたかった
    - https://speakerdeck.com/nazonohito51/whats-solid-principle

## Goとオブジェクト指向プログラミング
本題へ入る前に、`Go`とオブジェクト指向プログラミングの関係を考えてみます。
そもそもオブジェクト指向に準拠したプログラミング言語であることの条件とは何でしょうか。
さまざまな主張はありますが、本章では次の3大要素を備えることが「オブジェクト指向に準拠したプログラミング言語であること」とします。

 1. カプセル化（`Encapsulation`）
 2. 多態性（ポリモフィズム）（`Polymorphism`）
 3. 継承（`Inheritance`)


では`Go`はオブジェクト指向言語なのでしょうか。`Go`公式サイトには`Frequently Asked Questions (FAQ)`という「よくある質問と答え」ページがあります。
この中の`Is Go an object-oriented language?`（`Go`はオブジェクト指向言語ですか？）という質問に対する答えとして、次の公式見解が記載されています。

- Is Go an object-oriented language? | Frequently Asked Questions (FAQ)
    - https://golang.org/doc/faq#Is_Go_an_object-oriented_language

> Yes and no. Although Go has types and methods and allows an object-oriented style of programming, there is no type hierarchy. The concept of “interface” in Go provides a different approach that we believe is easy to use and in some ways more general. There are also ways to embed types in other types to provide something analogous—but not identical—to subclassing. Moreover, methods in Go are more general than in C++ or Java: they can be defined for any sort of data, even built-in types such as plain, “unboxed” integers. They are not restricted to structs (classes).
>
>Also, the lack of a type hierarchy makes “objects” in Go feel much more lightweight than in languages such as C++ or Java.

あいまいな回答にはなっていますが、「Yesであり、Noでもある。」という回答です。
`Go`はオブジェクト指向の3大要素を一部しか取り入れていないため、このような回答になっています。

## `Go`はサブクラシング（`subclassing`）に対応していない
多くの方がオブジェクト指向言語に期待する仕組みの1つとして、先ほど引用した回答内にもある`サブクラシング`（`subclassing`）が挙げられるでしょう。
もっと平易な言葉で言い直すと、`クラス（型）の階層構造（親子関係）による継承`です。
代表的なオブジェクト指向言語である`Java`でサブクラシングの例を書いたコードが次のコードです。
このコードは、親となる`Person`クラスと子となる`Japanese`クラスの定義と、`Person`クラスを引数に取るメソッドを含んでいます。

```java
class Person {
  String name;
  int age;
}

// Personクラスを継承したJapaneseクラス
class Japanese extends Person {
  int myNumber;
}

class Main {
  // Personクラスを引数にとるメソッド
  public static void Hello(Person p) {
    System.out.println("Hello " + p.name);
  }
}
```

`Person`クラスを継承した`Japanese`クラスのオブジェクトは、ポリモフィズムによって`Person`変数に代入できます。
また、同様に`Person`クラスのオブジェクトを引数にとるメソッドに対して代入することもできます。

```java
Japanese japanese = new Japanese();
person.name = "budougumi0617";
Person person = japanese;
Main.Hello(japanese);
```

このようなポリモフィズムを目的とした継承関係を表現するとき、`Go`ではインターフェースを使うでしょう。
しかし、`Go`は上記のような具象クラス（あるいは抽象クラス）を親とするようなサブクラシングによる継承の仕組みを言語仕様としてサポートしていません。

`Go`では埋込み（`Embedding`）を使って別の型に実装を埋め込むアプローチもあります。

- Embedding | Effective Go
    - https://golang.org/doc/effective_go.html#embedding

しかし、これは多態性や共変性・反変性を満たしません。

- 共変性と反変性 (C#)
    - https://docs.microsoft.com/ja-jp/dotnet/csharp/programming-guide/concepts/covariance-contravariance/

よって、埋め込みはオブジェクト指向で期待される継承ではなくコンポジションにすぎません。
次のコードは前述Javaのコードを`Go`で書き直したものです。
`Go`の例では、`Japanese`型のオブジェクトは`Hello`関数に利用することはできません。

```go
type Person struct {
  Name string
  Age  int
}

// Personを埋め込んだJapanese型。
type Japanese struct {
  Person
  MyNumber int
}

func Hello(p Person) {
  fmt.Println("Hello " + p.Name)
}
```

以上の例以外にも、`リスコフの置換原則`などの一部のSOLIDの原則はそのまま`Go`に適用することはできません。
しかし、SOLIDの原則のベースとなる考えを取り入れることでよりシンプルで可用性の高い`Go`のコードを書くことは可能です。

それでは、次節より`Go`のコードにSOLIDの原則を適用していくとどうなっていくか見ていきます。
なお、`Go`とSOLIDの原則については、`Dave Cheney`氏も2016年に`GolangUK`で`SOLID Go Design`というタイトルで発表されています。

- SOLID Go Design
    - https://dave.cheney.net/2016/08/20/solid-go-design

## [コラム]実装よりもコンポジションを選ぶ
あるクラスが他の具象クラス（抽象クラス）を拡張した場合の継承を、`Java`の世界では`実装継承`（`implemebtation inheritance`）と呼びます。
あるクラスがインターフェースを実装した場合や、インターフェースが他のインターフェースを拡張した場合の継承を`インターフェース継承`（`interface inheritance`）と呼びます。
`Go`がサポートしている継承は`Java`の言い方を借りるならば`インターフェース継承`のみです。
クラスの親子関係による`実装継承`はカプセル化を破壊する危険も大きく、深い継承構造はクラスの構成把握を困難にするという欠点もあります。
このことは代表的なオブジェクト指向言語である`Java`の名著、「`Effective Java`」の「`項目16 継承よりコンポジションを選ぶ`」でも言及されています（筆者が所有している`Effective Java`は第2版ですが、2018年に`Java 9`対応の`Effective Java 第3版`が発売されています）。
`Go`が`実装継承`をサポートしなかった理由は明らかになっていませんが、筆者は以上の危険性があるためサポートされていないと考えています。


---

転載は以上です。頒布物ではこの後個別の原則をGoでどう活かすかを20ページ弱にまとめています。

# おわりに
まだ私自身応援祭の具体的な形式がわかっておりませんが、ぜひご購入ください。
私自身日々学びながら設計・実装しているので、読後「これはこうじゃない？」みたいな感想などもいただけると幸いです。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://techbookfest.org/event/tbf08/circle/5752290824683520" data-iframely-url="//cdn.iframe.ly/ZldmKm4"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## 2020/02/28 21:00 追記
応援際の詳細が公開されました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">「<a href="https://twitter.com/hashtag/%E6%8A%80%E8%A1%93%E6%9B%B8%E5%85%B8?src=hash&amp;ref_src=twsrc%5Etfw">#技術書典</a> 応援祭」を3月7日から約1ヶ月間に渡って開催します！あり得ない短期間で工数をぶち込み、無理をおしてなんとか体制を整えました。すべては皆さんが一生懸命書いた技術書を、楽しみに待つ読者の元へ届けるためです。温かい気持ちで参加いただけたらうれしいです。<a href="https://t.co/yBgQxT97Q6">https://t.co/yBgQxT97Q6</a></p>&mdash; 技術書典公式アカウント (@techbookfest) <a href="https://twitter.com/techbookfest/status/1233246316584566784?ref_src=twsrc%5Etfw">February 28, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://blog.techbookfest.org/2020/02/28/cheering-tbf/index.html" data-iframely-url="//cdn.iframe.ly/JZFt5dJ?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# 関連情報
- [golang.tokyo](https://golangtokyo.github.io/)
- [技術書典 応援祭 golang.tokyoサークル詳細](https://techbookfest.org/event/tbf08/circle/5752290824683520)
- [2月29日、3月1日の技術書典8開催中止のお知らせ](https://blog.techbookfest.org/2020/02/16/tbf08-covid-19/)
