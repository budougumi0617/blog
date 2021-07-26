+++
title= "Terraformでimportを使う理由とfor_eachをつかったリソース定義に実リソースをimportする方法"
date= 2021-07-26T09:23:35+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["terraform"]
tags = ["terraform","import"]
keywords = ["terraform","import"]
twitterImage = "logos/terraform.png"
+++

`for_each`文を使ったTerraformのリソース定義に対してもimportができるよという話。  
なお、「どうしてそのようなことをする必要があるのか？」を説明する前置きのほうが長い。

<!--more-->

# TL;DR
- terraformで新規に定義を作るときは`terraform import`から始めるのが良い
- 複数の開発環境などで柔軟に利用できるリソースを定義する場合は`文を使う
    - https://www.terraform.io/docs/language/meta-arguments/for_each.html
- `for_each`文を使ったリソース定義に`import`するときは配列表記を使う

対応するterraformのバージョンは0.13以上ならば問題ないはず。2021年07月時点最新版の1.0.x系でも動作する。

# 凡人がTerraformで新規リソースを最速で定義する
Terraformで新しいリソースを作るときに最速な方法はなんだろうか？
「オレはPaaSのすべてのパラメータを理解している」というような凄腕でもない限り最速手順は手動で実リソースを作ってimportだ。
やはり調査したり編集しながらリソースを作る場合はコンソールなどから作る方が圧倒的に効率が良い。
詳細については以下のブログなどが参考になるだろう。

<div style="left: 0; width: 100%; height: 190px; position: relative;"><iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Ftech.layerx.co.jp%2Fentry%2Fimprove-iac-development-with-terraform-import" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>

# importしたリソース定義を使って他の環境にもリソースを作成していく

組織開発の中ではdevelopment/staging/pdoduction環境のようにクラウド上の構成を複数用意していることが多い。
そしてそれぞれの環境に対して共通のTerraform定義をapplyしていくことがほとんどだろう。
私の場合はmodule化してTerraformのリソース定義を各環境で共有している。
そのため前述の手順でリソースを作るときは次のような手順で行なう。

1. dev環境に実リソースをTerraformを使わず手動で作る
1. 実リソースをimportするためのTerraformリソースの定義をmoudleに作る
1. 手動で作ったリソースをmoduleに`terraform import`する
1. 各環境で`terraform apply`していく
    - dev環境では差分なし、他の環境では新規実リソースが生成される

この時必要になるテクニックとして、moduleには若干環境ごとに構成が異なる実リソースを作成できる拡張性をもたせておく。
よくやるのが次のようなオプションだ。

- 検証がしやすい様にdevだけワイルドカード的なルールをSecurityGroupに追加しておく
- dev/stgだけ踏み台サーバ、動作確認用のサーバからのアクセスを許可しておく

…などだ。この様な環境構成を設計するのに最適なのが`for_each`を使った構成だ。

## for_eachを使ったリソース定義
- https://www.terraform.io/docs/language/meta-arguments/for_each.html
- https://www.terraform.io/docs/language/functions/concat.html

`for_each`と`concat`を組み合わせると可変長引数を使っているかの様に動的な数のリソースを宣言できる。

下の例は変数の値によって作成される数が変わるセキュリティーグループルールのリソース定義だ。

- 必ず`default_group`からのssh接続を許可するルールが生成される
- `additional_ids`変数に要素があれば、その要素を許可するセキュリティーグループIDとしてルールが生成される

```hcl
variable "additional_ids" {
  description = "環境によって追加でsshを許可したいセキュリティーグループのIDリスト"
  type        = list(string)
}

# foo module内に定義されたリソースとする。
resource "aws_security_group_rule" "dynamic_rules" {
  for_each = toset(concat([aws_security_group.default_group.id], var.additional_ids))

  security_group_id        = aws_security_group.target_group.id
  type                     = "ingress"
  description              = "allow access by sg id"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = each.key
}
```

`for_each`に与える要素はSetかMapの必要がある。またリストのマージをスマートにできないので少し冗長に書いてある。

では`for_each`文を使ってこのようなTerraformの定義を用意した場合、手動作成した実リソースを冒頭に書いたようにこの定義へimportするにはどうしたら良いのだろうか？

# キーの名前をつかって配列アクセスのようにimportする

importする際には基本的にリソース名が必要になる。`for_each`を使ったリソースの場合、`for_each`によって生成されるキーがリソース名の一部になる。これを使ってimportを行えば良い。  
たとえば`foo module`の`aws_security_group_rule.dynamic_rules`から生成されるリソースは`aws_security_group.target_group.id`が`"sg-hogeohge1"`だった場合、
`module.foo.aws_security_group_rule.dynamic_rules["sg-hogeohge1"]`というようなリソース名になる。
よって先ほど例にあげた`aws_security_group_rule`をimportする場合は次のようになる。

- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule

```bash
# target_group sgのidがsg-foofoo1、default_group sgのidがsg-hogeohge1だった場合
$ terraform import \
    'module.foo.aws_security_group_rule.dynamic_rules["sg-hogeohge1"]' \
    sg-foofoo1_ingress_tcp_443_443_sg-hogehoge1
```


# 終わりに
背景の説明が長くなったが、`for_each`を使った定義でimportする方法を紹介した。  
Terraformの`for_each`は`for_each`文の外に置かれたリソースが`for_each`で増えるので通常のプログラム言語のループのスコープで考えると難しい。

完全に余談だが、`aws_security_group_rule`はsecurity group rule idでimportしたいんだけれどaws cliが対応していないのだろうか？
