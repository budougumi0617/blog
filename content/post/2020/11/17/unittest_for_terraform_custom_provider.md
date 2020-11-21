+++
title= "自作Terraform Providerのユニットテストの書き方"
date= 2020-11-17T09:30:48+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["terraform","go"]
tags = ["golang","terraform","test"]
keywords = ["Go","unittest","terraform","provider"]
twitterImage = "logos/terraform.png"
+++

公式チュートリアルには載っていなかったので、自作Terraform Providerを作るときのユニットテストの書き方をメモしておく。  
なお、最初にコメントしておくと今回の記事はかなり説明を省略しているので各Providerにコミットしたことがあるか自作Providerを作った人じゃないとわからなそう…

<!--more-->

# TL;DR
- TerraformはProvider経由で各種サービスのリソースを操作する
  - https://www.terraform.io/docs/providers/index.html
- SDKを使えば自作Providerを作ることも可能
  - https://github.com/hashicorp/terraform-plugin-sdk
- 公式ガイドに載っていないが、`schema.Resource#TestResourceData`メソッドを使うとテストが書きやすい
  - https://pkg.go.dev/github.com/hashicorp/terraform-plugin-sdk/v2@v2.2.0/helper/schema#Resource.TestResourceData
- ユニットテストを書けば動作確認も簡単にできるので開発がはかどる！

```go
// schema.Resource.ReadContextに設定する関数のテスト
func Test_dataSourceYourObjectRead(t *testing.T) {
  // *schema.ProviderData.ConfigureContextFuncの戻り値にしているオブジェクト
  m := providerConfigureReturnedObject
  // *schema.Provider.DataSourcesMapに設定するResource
  d := dataSourceYourObject().TestResourceData()

  got := dataSourceYourObjectRead(context.TODO(), d, m)
  if diff := cmp.Diff(got, tt.want); diff != "" {
    t.Errorf("dataSourceYourObjectRead: (-got +want)\n%s", diff)
  }
}
```

# Terraform Providerの概要
Terraformはプロパイダーを経由して各種リソースを操作する。
AWSプロパイダーやGCPプロパイダー経由でマネージドサービスを管理している方も多いだろう。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/hashicorp/terraform-provider-aws" data-iframely-url="//cdn.iframe.ly/W3cxksd"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/hashicorp/terraform-provider-google" data-iframely-url="//cdn.iframe.ly/WBrqvjN"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

TerraformはIaaS以外もプロパイダーさえあれば宣言的に管理できる。
公式で提供されているプロパイダーだけでもこれだけある。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.terraform.io/docs/providers/index.html" data-iframely-url="//cdn.iframe.ly/6f0zAr6"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

また、Goが書けるならばSDKを使って自作のプロパイダーを作成し、レジストリに公開することもできる、

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/hashicorp/terraform-plugin-sdk" data-iframely-url="//cdn.iframe.ly/d5HNUnD"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


自作プロパイダーは次の公式ガイドを読みながら実装することができる。
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://learn.hashicorp.com/collections/terraform/providers" data-iframely-url="//cdn.iframe.ly/lHR2l7J?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


日本語資料がよいならソフトウェアデザイン11月号の「作品で魅せるGoプログラミング」に解説が載っている。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B08KSQ2FB5&linkId=28c90a4791e5ef6496c909b3c558908f"></iframe>

# 自作Provider実装時の動作確認のめんどくささ
ホビープロジェクトとしてTerraformの自作Providerを作っている。  
Providerは`terraform`コマンドが呼び出すサププロセスとなる。  
そのため、作るのは簡単だが動作確認がめんどくさい。  
さきほど紹介した資料ではどちらにも一度ビルドしてTerraform経由で起動する動作確認方法が記載されている。  

- https://learn.hashicorp.com/tutorials/terraform/provider-setup?in=terraform/providers#test-the-provider

```bash
$ pwd
/Users/budougumi0617/go/src/github.com/budougumi0617/terraform-provider-hashicups
# 一度ビルドする
$ make install
go build -o terraform-provider-hashicups
mv terraform-provider-hashicups ~/.terraform.d/plugins/hashicorp.com/edu/hashicups/0.2/darwin_amd64
# tfファイルを用意したディレクトリに移動する
$ cd examples
# ビルドしたら必ずinitしないといけない
$ terraform init
# ここまでやってやっとapplyして動作確認ができる
$ terraform apply --auto-approve
```

この方法だと毎回ビルドをしたあと異なるディレクトリで`terraform init && terraform apply`をする必要があり時間もかかる。  
また、デバッガのアタッチも難しい。そのためユニットテストで動作確認をしたい。

# 自作Providerのユニットテストを書く

## 自作Providerの実装構造
Terraformで自作プロバイダーを実装する流れはざっくり次の通り。

- データあるいはリソースの構造を定義する
- データあるいはリソースに対して、CRUD操作にひとつひとつ対応する関数を実装する

CRUD操作に対応するそれぞれの関数は次のシグネチャを持つ関数として実装する。  

- [github.com/hashicorp/terraform-plugin-sdk/helper/schema/resource.go](https://github.com/hashicorp/terraform-plugin-sdk/blob/6e57e60d8383a26ab179b1b142b3f6ca452229c0/helper/schema/resource.go#L226-L236)


```go
// See Resource documentation.
type CreateContextFunc func(context.Context, *ResourceData, interface{}) diag.Diagnostics

// See Resource documentation.
type ReadContextFunc func(context.Context, *ResourceData, interface{}) diag.Diagnostics

// See Resource documentation.
type UpdateContextFunc func(context.Context, *ResourceData, interface{}) diag.Diagnostics

// See Resource documentation.
type DeleteContextFunc func(context.Context, *ResourceData, interface{}) diag.Diagnostics
```

（つまり同じシグネチャなのだが、）この関数の引数にわたすオブジェクトを用意すれば各`XxxxContextFunc`に対応するCRUD操作のユニットテストが実行できる。

## テスト対象の自作データの定義

今回テストの対象になるデータリソースの定義を確認する。

```go
import (
  "context"
  "fmt"

  pixela "github.com/ebc-2in2crc/pixela4go"

  "github.com/hashicorp/terraform-plugin-sdk/v2/diag"
  "github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
)

func dataSourceGraphs() *schema.Resource {
  return &schema.Resource{
    ReadContext: dataSourceGraphsRead,
    Schema: map[string]*schema.Schema{
      "graphs": {
        Type:     schema.TypeList,
        Computed: true,
        Elem: &schema.Resource{
          Schema: map[string]*schema.Schema{
            "id": {
              Type:     schema.TypeString,
              Computed: true,
            },
            "name": {
              Type:     schema.TypeString,
              Computed: true,
            },
            // ...
          },
        },
      },
    },
  }
}

// この関数をテストしたい
func dataSourceGraphsRead(
  ctx context.Context, d *schema.ResourceData, m interface{},
  ) diag.Diagnostics {
  client := m.(*pixela.Client)
  var diags diag.Diagnostics
  // Do anything...
  return diags
}
```

また、Providerの`ConfigureContextFunc`に登録している関数は次の通り。

```go
import (
  "context"
  "fmt"

  pixela "github.com/ebc-2in2crc/pixela4go"
  "github.com/hashicorp/terraform-plugin-sdk/v2/diag"
  "github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
)

func providerConfigure(
  _ context.Context, d *schema.ResourceData,
  ) (interface{}, diag.Diagnostics) {
  un := d.Get("username").(string)
  if un == "" {
    return nil, diag.FromErr(fmt.Errorf("not find username"))
  }

  token := d.Get("token").(string)
  if token == "" {
    return nil, diag.FromErr(fmt.Errorf("not find token"))
  }

  return pixela.New(un, token), diag.Diagnostics{}
}
```

`dataSourceGraphsRead`関数をテストするには、`*schema.ResourceData`オブジェクトを入手する必要がある。  
`*schema.ResourceData`オブジェクトは、`dataSourceGraphs`関数の戻り値である`*schema.Resource`オブジェクトから入手することができる。

- [schema.Resource#TestResourceData](https://pkg.go.dev/github.com/hashicorp/terraform-plugin-sdk/v2@v2.2.0/helper/schema#Resource.TestResourceData)


> func (*Resource) TestResourceData  
> `func (r *Resource) TestResourceData() *ResourceData`  
> TestResourceData Yields a ResourceData filled with this resource's schema for use in unit testing
>
> TODO: May be able to be removed with the above ResourceData function.

第1引数のcontextは適当なcontextでよく、第3引数は`ConfigureContextFunc`に登録する`providerConfigure`関数の戻り値で良い。  
今回の場合だと第3引数は`providerConfigure`関数が返す`*pixela.Client`オブジェクトになる。

これらを踏まえると、次のようにtestを書くことができる。

```go
package pixela

import (
  "context"
  "os"
  "testing"

  pixela "github.com/ebc-2in2crc/pixela4go"
  "github.com/google/go-cmp/cmp"
  "github.com/hashicorp/terraform-plugin-sdk/v2/diag"
)

func Test_dataSourceGraphsRead(t *testing.T) {
  tests := [...]struct {
    name string
    want diag.Diagnostics
  }{
    {
      name: "confirmClientResponse",
      want: nil,
    },
  }
  for _, tt := range tests {
    usename := os.Getenv("PIXELA_USERNAME")
    token := os.Getenv("PIXELA_TOKEN")
    if usename == "" || token == "" {
      t.SkipNow()
    }
    t.Run(tt.name, func(t *testing.T) {
      m := pixela.New(usename, token)
      d := dataSourceGraphs().TestResourceData()
      got := dataSourceGraphsRead(context.TODO(), d, m)
      if diff := cmp.Diff(got, tt.want); diff != "" {
        t.Errorf("dataSourceGraphsRead: (-got +want)\n%s", diff)
      }
    })
  }
}
```

これでunittestとして動作確認ができるようになった。
また、デバッガを接続して動作確認もできるので、データの流れ方も確認することができる。

# 終わりに
前提知識の補足をほとんど書いていないが、terraformのProviderのユニットテストを書く方法をまとめた。  
terraform-provider-awsなどを確認すると、もっとテスト用の関数を多用しているので、他にもどんなテストが書けるか読んでおきたい。

# 参考
- https://github.com/hashicorp/terraform-plugin-sdk
- Providers - Terraform by Hashicorp
  - https://www.terraform.io/docs/providers/index.html
- Call APIs with Terraform Providers | Terraform - Hashicorp Learn
  - https://learn.hashicorp.com/collections/terraform/providers
- https://github.com/hashicorp/terraform-provider-aws


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B08KSQ2FB5&linkId=28c90a4791e5ef6496c909b3c558908f"></iframe>