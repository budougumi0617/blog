+++
title= "TerraformでPixelaのグラフを宣言できるProviderを作った #pixela"
date= 2020-12-11T09:06:15+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go","terraform"]
tags = ["terraform","golang","pixela","oss"]
keywords = ["terraform","acceptance test","pixela"]
twitterImage = "logos/terraform.png"

+++


[Pixela](https://pixe.la/ja)のTerraform Providerを作った。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/terraform-provider-pixela" data-iframely-url="//cdn.iframe.ly/zjuslmC"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>



<!--more-->

# TL;DR
- PixelaはGitHubのContributions風のグラフを生成するWebサービス
    - https://pixe.la
- PixelaのTerraform Providerを作った
    - https://github.com/budougumi0617/terraform-provider-pixela
- 使い方の紹介
- 実装の際にハマったところ
- 次はSDKも自作したい
- 他にもAPIがあったらTerraform Provider書こうかな


```hcl
terraform {
  required_providers {
    pixela = {
      versions = ["0.0.5"]
      source   = "github.com/budougumi0617/pixela"
    }
  }
}

provider pixela {
  // Pixelaに登録したユーザー名
  username = "budougumi0617"
}

resource "pixela_graph" "sample" {
  graph_id              = "sample"
  name                  = "update from terraform"
  unit                  = "page"
  type                  = "int"
  color                 = "ajisai"
  timezone              = "Asia/Tokyo"
  self_sufficient       = "none"
  is_secret             = true
  publish_optional_data = false
}
```


# Pixelaとは
Pixelaは[@a-know](https://twitter.com/a_know)さんが作っているGitHubのContributions風のグラフを生成するWebサービスだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://pixe.la" data-iframely-url="//cdn.iframe.ly/q3gtB1F?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

APIを手順通りに叩けば、任意のデータを記録できるグラフを生成できる。

![https://pixe.la/v1/users/budougumi0617/graphs/egiu](https://pixe.la/v1/users/budougumi0617/graphs/egiu)
（これは休止している私の英語学習記録グラフ…）

また、PixelaはREST風のAPIを使って操作するのが基本で、APIドキュメントも公開されている。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fdocs.pixe.la%2Fentry%2Fapi-index" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

# Terraform Providerとは
Terraformは宣言的にリソースを管理するためのツールだが、TerraformはProviderと呼ばれるプラグイン経由で各リソースを操作している。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.terraform.io/docs/providers/index.html" data-iframely-url="//cdn.iframe.ly/6f0zAr6"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

たとえば、AWSだったらTerraformはAWS Providerを使ってAWS上のリソースを操作している。
TerrformはSDKが公開されているので、自分で好きなProviderを実装し、Terraform registoryに公開することができる。

# Terraform Provider Pixela
リソースがあって、CRUDのAPIが用意されていればTerraform Providerをつくることができる。
こうして今回つくったのがPixela Providerだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/terraform-provider-pixela" data-iframely-url="//cdn.iframe.ly/zjuslmC"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

Registryにも公開済みだ。

- https://registry.terraform.io/providers/budougumi0617/pixela

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://twitter.com/hashtag/pixela?src=hash&amp;ref_src=twsrc%5Etfw">#pixela</a> を宣言的に管理するぞ！ということで初めてTerraform Providerを作ってみた！<br>v0.0.4ではグラフをTerraformで定義できます！<a href="https://t.co/LdD99leEjO">https://t.co/LdD99leEjO</a></p>&mdash; Yoichiro Shimizu (@budougumi0617) <a href="https://twitter.com/budougumi0617/status/1331034752782987264?ref_src=twsrc%5Etfw">November 24, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 使い方
使いかたはTerraformを使ったことがあれば簡単で次のような宣言でグラフを生成できる。
なお、`terraform apply`するとき、`PIXELA_TOKEN`環境変数にトークンが設定されている必要がある。

```hcl
terraform {
  required_providers {
    pixela = {
      versions = ["0.0.5"]
      source   = "github.com/budougumi0617/pixela"
    }
  }
}

provider pixela {
  // Pixelaに登録したユーザー名
  username = "budougumi0617"
}

resource "pixela_graph" "sample" {
  graph_id              = "sample"
  name                  = "update from terraform"
  unit                  = "page"
  type                  = "int"
  color                 = "ajisai"
  timezone              = "Asia/Tokyo"
  self_sufficient       = "none"
  is_secret             = true
  publish_optional_data = false
}
```


# 技術的な話
Terraform Providerは実装からテスト、リリースまでドキュメントを読みながら実装すれば簡単に作ることができる。  
参考ドキュメントと少しハマったところを紹介しておく。

## Terraform Providerの作り方
Providerの作り方は公式ガイドを参考にしながら作ることができる。

- Call APIs with Terraform Providers
    - https://learn.hashicorp.com/collections/terraform/providers

また、次の雑誌でも日本語でProviderの作り方が公開されている。
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B08KSQ2FB5&linkId=b53d36b87ef5ae52ae71430a52a3b258"></iframe>


## リリースの仕方
Terraform Providerを作ったらレジストリにも登録することができる。
こちらも手順通りにやればよい。

- https://www.terraform.io/docs/registry/providers/publishing.html


## テストの書き方
Terrformは受け入れテスト（実際にリソースを作って確認するテスト）もSDKで簡単に作ることができる。  
次のドキュメントを読みながら実装することができた。

- Acceptance Tests
    - https://www.terraform.io/docs/extend/testing/acceptance-tests/index.html
- Testing Patterns
    - https://www.terraform.io/docs/extend/best-practices/testing.html

ユニットテストは次のように書くことができる。

- [自作Terraform Providerのユニットテストの書き方](/2020/11/17/unittest_for_terraform_custom_provider/)

## 少しハマったところ
Goをやっていれば型を使って実装をするだろう。  
Terrform Providerを実装するさいは型（構造体）を使って結果を取得しても、そのあとは構造体フィールドをひとつひとつセットしないといけないようだった。


```go
func resourceGraphRead(_ context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	// Warning or errors can be collected in a slice type
	var diags diag.Diagnostics
	client := m.(*pixela.Client)

	g, err := client.Graph().Get(&pixela.GraphGetInput{ID: pixela.String(d.Id())})
	if err != nil {
		return diag.FromErr(err)
	}
	if err := d.Set("graph_id", g.ID); err != nil {
		return diag.FromErr(err)
	}
	if err := d.Set("name", g.Name); err != nil {
		return diag.FromErr(err)
	}
	if err := d.Set("type", g.Type); err != nil {
		return diag.FromErr(err)
    }
    // ...
```

また、レジストリ登録時に少しハマったが、ちゃんと手順通りにやっていれば問題ない（完全に私が悪かった）。

- [TerraformレジストリにリリースしようとGPG Keyをimportしようとすると「Could not find valid encryption key packet in key ~~」とエラーになる](/2020/11/27/terraform_could_not_find_valid_encryption_key_packet_in_key/)

## Special Thanks...
1.20.1以前のPixelaはグラフ単体のJSON情報を取得することができなかった。  
なので[@a-know](https://twitter.com/a_know)さんにリクエストしてエンドポイントを増やしてもらった。大感謝…

https://github.com/a-know/Pixela/releases/tag/v1.21.0

# 終わりに
PixelaをTerrformで使うためのProviderを作った。ProviderはPixelaのAPI操作に3rdパーティのライブラリを使っているのだが、少し使い方が合わないのでも自作ライブラリを作って差し替える予定。  
グラフにプロットするPixel自体もサポートしようかなとと思うが、毎回terraformでプロット打つのはちょっと違うかなと思っているので、Provider自体はどうやって機能拡張しようか考えている。

他にもteraformサポート外のAPIを見つけたらProviderを書いてみたい。

# 参考
- https://github.com/budougumi0617/terraform-provider-pixela
- https://registry.terraform.io/providers/budougumi0617/pixela
- https://pixe.la/ja
- https://learn.hashicorp.com/collections/terraform/providers

# 関連
- [TerraformレジストリにリリースしようとGPG Keyをimportしようとすると「Could not find valid encryption key packet in key ~~」とエラーになる](/2020/11/27/terraform_could_not_find_valid_encryption_key_packet_in_key/)
- [自作Terraform Providerのユニットテストの書き方](/2020/11/17/unittest_for_terraform_custom_provider/)

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07XT7LJLC&linkId=00bdffd3d651b7d204a542195114cba3"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B08KSQ2FB5&linkId=b53d36b87ef5ae52ae71430a52a3b258"></iframe>