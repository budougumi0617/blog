+++
title= "[Re:VIEW] textlintのlintエラーからttインライン命令で装飾された文字列を除外する"
date= 2020-01-19T11:46:11+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["textlint", "review"]
tags = ["textlint", "review"]
keywords = ["Re:VEIW", "文書校正", "textlint", "技術書典"]
twitterImage = "logos/textlint.png"
+++



Re:VIEWで文書を作成するときはtextlintを使って文書校正をしている。  
`@<tt>{git commit --amend}`のような等幅設定をしている中で、`git => Git (prh)`というような警告を受けたくないときに行なう設定をまとめる。

<!--more-->

# TL;DR
- `tt`インライン命令で囲った文字列は静的解析の対象にしたくない
  - `@<tt>{git commit --amend}`の"git"はコマンド名なので、"Git`ではない
- Re:VIEWファイルにtextlintを使っているときは、`textlint-plugin-review`プラグインを使っているだろう。
  - https://github.com/orangain/textlint-plugin-review
- `textlint-plugin-review`プラグインでは、`tt`インライン命令は`Teletype`というSyntaxの分類にパースされる。
  - `tt:      inlineTextTagParser(Syntax.Teletype)`
- `Teletype`はtextlint本体では`Inline`というノードタイプに分類される。
  - `Teletype: 'Inline'`
- `textlint-filter-rule-node-types`プラグインを使って、`Inline`を静的解析対象から除外すればよい。
  - https://github.com/textlint/textlint-filter-rule-node-types
- `code`インライン命令で装飾された文字列は警告から除外されているようだ

今回書いた`.textlintrc`ファイルの設定は以下の通り。
```json
// .textlintrc
{
  "filters": {
    // ...
    "node-types": {
      "nodeTypes": ["Inline"]
    }
  }
}
```

# textlintが等幅フォント文字列（ttインライン命令）に対して静的解析エラーを出力してしまう

技術書典に向けてRe:VIEW形式で原稿を書くにあたり、textlintを導入している。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/kmuto/review" data-iframely-url="//cdn.iframe.ly/ydUD1Nk"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

textlintは文書に対して静的解析を行なってくれるツールだ。プラグインでさまざまなルールを追加できて、文書校正を自動で行なってくれる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/textlint/textlint" data-iframely-url="//cdn.iframe.ly/uC7J82H"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Re:VIEWではインライン命令を使って文書の装飾をする。
コードや技術的用語を文中で書くときは等幅フォントにするため、`@<code>`インライン命令や`@<tt>`インライン命令を使う。

- [https://github.com/kmuto/review/blob/master/doc/format.ja.md#書体][shotai]


[shotai]: https://github.com/kmuto/review/blob/master/doc/format.ja.md#%E6%9B%B8%E4%BD%93

```
@<code>{golang.org/x}という@<code>{import}パスから始まる準標準パッケージがあることをご存じでしょうか。
アセンブラ別の実装が含まれており、Goのデバッガである@<tt>{Delve}（@<tt>{dlv}コマンド）@<fn>{delve}などで利用されています。
```

`@tt`インライン命令で囲った文字列はコマンドだったりするので、textlintで静的解析の対象にしてほしくない。

```bash
$ cat articles/budougumi0617.re
@<code>{git commit --amend}はエラーにしてほしくない。（デフォルトで無視される）
@<tt>{git commit --amend}はエラーにしてほしくない。

$ $(npm bin)/textlint articles/*.re

/Users/budougumi0617/go/src/github.com/FOOOO/SHOTEN8/articles/budougumi0617.re
  2:7  ✓ error  git => Git  prh

✖ 1 problem (1 error, 0 warnings)
✓ 1 fixable problem.
Try to run: $ textlint --fix [file]

```

上記の例の場合は、`git`コマンドに対して、（正式名称の）`Git`にしろと静的解析エラーが発生している。  
`@<code>`インライン命令を使えば解決するのだが、せっかくなので誤検知を解消することにした。

# ttインライン命令がtextlintでどのように認識されているか
Re:VIEW形式の文書をtextlintを静的解析をするときは、`textlint-plugin-review`プラグインを使っているだろう。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/orangain/textlint-plugin-review" data-iframely-url="//cdn.iframe.ly/0kWQW7r"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

`textlint-plugin-review`プラグインがどのように`Re:VIEW`形式の文書から抽象構文木（AST）を作っているのか確認してみる。  
`textlint-plugin-review`プラグインでは、tt命令は`Teletype`という`Syntax.Teletype`の分類にパースされる。

- [https://github.com/orangain/textlint-plugin-review/src/inline-parsers.js][inline-parsers]
[inline-parsers]: https://github.com/orangain/textlint-plugin-review/blob/7d660f45a6a81a3456059ed7dfc598df253a2ca8/src/inline-parsers.js

```js
const InlineParsers = {
  // text tags
  
  // ...
  
  tt:      inlineTextTagParser(Syntax.Teletype),
  
  // ...
```

`Syntax.Teletype`がtextlint本体ではどういうノードとして扱われるかというと、`Inline`というノードタイプに分類される。

- [https://github.com/orangain/textlint-plugin-review/src/mapping.js][mapping]
[mapping]: https://github.com/orangain/textlint-plugin-review/blob/7d660f45a6a81a3456059ed7dfc598df253a2ca8/src/mapping.js
```js
export const Syntax = {
  // human readable name in ReVIEW's context => textlint name

  // ...

  // ReVIEW specific inline tags
  // NOTE: 'Inline' means review's inline tag having no special meanings, whose children are Strs.
  //       'Reference' means reference to other tag, which have no child.
  //       'NonString' means non-string stuffs like character number, equation, etc.
  Teletype: 'Inline',
  
  // ...
};
```

よって、`Inline`ノードを静的解析のルールから除外すれば良い。

# textlint-filter-rule-node-typesプラグインを使ってInlineノードを除外する
textlintにおいて、特定のノードを静的解析のルールから除外するには、`textlint-filter-rule-node-types`プラグインを使えばよい。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/textlint/textlint-filter-rule-node-types" data-iframely-url="//cdn.iframe.ly/fg9QbXW"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

``textlint-filter-rule-node-types`プラグインを`npm install`したあと、`.textlintrc`を以下のようにすると、`Inline`ノードに対して静的解析が行われなくなった。


```json
// .textlintrc
{
  "rules": {
    // ...
  },
  "plugins": [
    "review"
  ],
  "filters": {
    // ...
    "node-types": {
      "nodeTypes": ["Inline"]
    }
  }
}
```

# 終わりに
（textlintやJavaScriptにはまったく関係はないが、）最近パーサーを書いているので、`textlint-plugin-review`プラグインの中の構造もなんとなく読むことができた。  
何事も勉強したことは無駄にならない。

# 参考
- https://github.com/kmuto/review/
- https://github.com/textlint/textlint
- https://github.com/orangain/textlint-plugin-review
- https://github.com/textlint/textlint-filter-rule-node-types
