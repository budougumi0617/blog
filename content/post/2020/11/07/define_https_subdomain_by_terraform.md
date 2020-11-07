+++
title= "TerraformでAWS上にHTTPS化したサブドメインを定義する"
date= 2020-11-07T00:19:31+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["terraform","aws"]
tags = ["terraform","aws","route53","https"]
keywords = ["aws","terraform","route53","https"]
twitterImage = "logos/terraform.png"
+++


個人のAWS環境でTerraformを使ってHTTPS化したサブドメインを定義した。  
普段なかなかしないことで忘れてしまうので手順をまとめおく。

<!--more-->

# TL;DR
- Terraformを使ってAWS上でHTTPS化したサブドメインを構成したい
- ルートドメインのホストゾーンをTerrformで作っても登録済みドメインのネームサーバの設定は手動になるので注意する
- ワイルドカード付きの証明書をTerffaformで生成するときは少しテクニックが必要になる
- ハマりどころを解決できれば少々の定義で構築できた

なお、本記事で利用しているTerraformとAWS Providerのバージョンは以下となる。

```hcl
terraform {
  required_version = ">= 0.13.4"
}

provider "aws" {
  version = "~> 3.6.0"
}
```

# HTTPS化したサブドメインを作成したい
この記事の前提とやりたいことは次の通り。

- 前提条件
    - example.com というドメインをRoute53の登録済みドメインとして登録済み
- example.comまたは*.example.comをHTTPS化するSSL証明書を発行する
- https://subdomain.example.com というようなHTTPS化されたサブドメインを定義する。

本記事で利用するAWSのサービスはRoute53とAWS Certificate Manager（ACM）だ。  
Route53はご存知の通りAWS上でドメインを使ってルーティングを構成するDNSサービスだ。  
ACMはSSL/TLS証明書を管理するサービスで、今回はHTTPS通信に利用する証明書を自動管理させる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://aws.amazon.com/jp/route53/" data-iframely-url="//cdn.iframe.ly/MhwPNuE?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://aws.amazon.com/jp/certificate-manager/" data-iframely-url="//cdn.iframe.ly/Ih512Ce?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# Route53でホストゾーンを定義する
まずRoute53でsubdomain.example.com（サブドメイン）をホストゾーンで定義する。

- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_zone

なお、Route53でexample.comを登録済みドメインにしている場合、すでにホストゾーンが作成されているはずである。  
Terraform上では、既成のexample.com（ルートドメイン）に対応するホストゾーンをデータソースで参照する。

```hcl
data aws_route53_zone example_com {
  name = "example.com"
}

resource aws_route53_zone subdomain_example_com {
  name = "subdomain.example.com"
}
```

**ルートドメインをTerraformで定義しようとしないこと**。

## ルートドメインのホストゾーンはTerraformで作成しないこと
AWS Provider 3.6.0系でもRoute53の「登録済みドメイン」をTerraformで管理することはできない。  
Terraformでルートドメインを定義した場合、定義したルートドメインに割り当てられるNSコード（ネームサーバ）のIPと、登録済みドメインのネームサーバが一致しなくなるので証明書のDNS認証などができなくなる。

- Route 53 に登録されているドメインのホストゾーンの置き換え
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/domain-replace-hosted-zone.html

> ホストゾーンを作成する場合、Route 53 はホストゾーンに一連の 4 つのネームサーバーを割り当てます。ホストゾーンを削除して新しいゾーンを作成すると、Route 53 は別の一連の 4 つのネームサーバーを割り当てます。通常、新しいホストゾーンのネームサーバーは、以前のホストゾーンのネームサーバーと一致しません。新しいホストゾーンのネームサーバーを使用するようにドメイン設定を更新しないと、ドメインはインターネット上で利用できなくなります。


# サブドメインとルートドメインのホストゾーンを関連付ける
次に、作成するサブドメインをルートドメインに関連付ける。  
具体的には、ルートドメイン (example.com) のホストゾーンにサブドメインのホストゾーン割り当てられる4つのネームサーバーをNSレコードとして登録する。

- サブドメインのトラフィックのルーティング
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/dns-routing-traffic-for-subdomains.html

TerraforrmでNSレコードを作成するコードは次の通り。

- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record

```hcl
resource aws_route53_record ns_record_for_subdomain {
  name    = aws_route53_zone.example_com.name
  zone_id = data.aws_route53_zone.example_com.id
  records = [
    aws_route53_zone.subdomain_example_com.name_servers[0],
    aws_route53_zone.subdomain_example_com.name_servers[1],
    aws_route53_zone.subdomain_example_com.name_servers[2],
    aws_route53_zone.subdomain_example_com.name_servers[3]
  ]
  ttl  = 300
  type = "NS"
}
```

これでリクエストの流れはできた。

# ACMでHTTPS化用のSSL証明書を作成する
次にHTTPS化の準備をする。冒頭で述べた通りACMを使って証明書を発行する。  
AWSでHTTPS用の証明書を発行する場合、DNS認証を使っておけば証明書の自動更新が実現できる。  
当然今回もこれを利用する設定を行う。

- DNSを使用したドメイン所有権の検証
    - https://docs.aws.amazon.com/ja_jp/acm/latest/userguide/gs-acm-validate-dns.html
- 新しいドメインの DNS ルーティングの設定
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/dns-configuring-new-domain.html


## ワイルドカード付きの証明書を発行する
ルートドメイン、サブドメインすべてをHTTPS化したいため、ワイルドカードを含んだSSL証明書を発行する必要がある。

まず、SSL証明書はの定義は次の通り。`subject_alternative_names`を使うことでサブドメインもSSL証明書の対象に含めることができる。

- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/acm_certificate

```hcl
resource aws_acm_certificate example_com {
  domain_name               = data.aws_route53_zone.example_com.name
  subject_alternative_names = [format("*.%s", data.aws_route53_zone.example_com.name)]
  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }
}
```

`format`関数はTerraformの標準関数のひとつで、ざっくりいうとGoの`fmt.Printf`のように文字列を生成できる。

- format Function
    - https://www.terraform.io/docs/configuration/functions/format.html

証明書の検証につかうレコードはこのように定義する。

```hcl
resource aws_route53_record certificate {
  for_each = {
    for dvo in aws_acm_certificate.example_com.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.example_com.id
  allow_overwrite = true
}
```

最後にapply時SSL証明書の検証が完了するまで待機する設定を書いておく。


- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/acm_certificate_validation

```hcl
resource aws_acm_certificate_validation cert {
  certificate_arn         = aws_acm_certificate.example_com.arn
  validation_record_fqdns = [for record in aws_route53_record.certificate : record.fqdn]
}
```

## terraform apply中に「Tried to create resource record set....but it already exists」が出て失敗する
ワイルドカードを使った証明書を作成しようとすると、ワイルドカードとルートドメインの検証用レコードが同じ内容になるためエラーが発生する。  
**これを回避するには`allow_overwrite`を`true`にしておくのを忘れないこと**。

本記事に記載した先ほどの`aws_route53_record certificate`はAWS Provider v3の定義方法だ。  
AWS Provider v2などで同様の現象を回避したい場合は次のブログが参考になる。

- SANsにワイルドカードが入ったACMのDNS認証なSSL証明書をTerraformで作るときのハマりどころ
    - https://fusagiko.hatenablog.jp/entry/2019/09/30/223700

## DNS認証が終わらない
正しく設定できているように見えるのにDNS認証が通らない場合はネームサーバの設定がおかしく、検証用リクエストがさばけていない可能性がある。  
前述したとおり、ルートドメインをTerraformで新しく作った場合などに発生する。  
ルートドメインのホストゾーンのNSレコードに記載されているネームサーバのIPを登録済みドメインのネームサーバのIPとして登録すること。

- ドメインのネームサーバーおよびグルーレコードの追加あるいは変更
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/domain-name-servers-glue-records.html


# ALBへ通信を流す
ここまでできればあとはサブドメインのホストゾーンからALBへ通信を流し込むAレコードを作成すればよい。


```hcl
resource "aws_route53_record" "subdomain_example_com" {
  zone_id = aws_route53_zone.subdomain_example_com.zone_id
  name    = aws_route53_zone.subdomain_example_com.name
  type    = "A"

  # 実際に通信をさばくALBの情報
  alias {
    name                   = aws_lb.xxx.dns_name
    zone_id                = aws_lb.xxx.zone_id
    evaluate_target_health = true
  }
}
```

これで、 `https://subdomain.example.com`の通信がALBへ流し込まれるようになる。

AレコードはAWS独自拡張DNSレコードタイプのエイリアスレコードのことだ（よくわかっていないが速いらしい）。

- サポートされる DNS レコードタイプ
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/ResourceRecordTypes.html
- エイリアスレコードと非エイリアスレコードの選択
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html

TerraformのApplyが終わればローカルから`https://subdomain.example.com`への通信が可能になっているはずだ（当然ALB以降に何かしらレスポンスする定義がないとダメだが）。

# 最後に
私はアプリケーションエンジニアなので、普段ALBらへんまではTerraformで書いたり、覗いたことがあった。  
今回いつもSREの方々に任せているところを自分ではじめてRoute53やACMを自分で設定してみてネットワークについて少し理解が深まった。  
ただ、アプリケーションを作ってObservabilityの勉強をしたいのが目的なので、ゴール（スタート？）はまだまだ遠い…

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07XT7LJLC&linkId=f7b69fcc728d07bb0be7d26f22163b8a"></iframe>


最後に今回使ったサンプルコードをまとめておく。

```hcl
terraform {
  required_version = ">= 0.13.4"
}

provider "aws" {
  version = "~> 3.6.0"
}

data aws_route53_zone example_com {
  name = "example.com"
}

resource aws_route53_zone subdomain_example_com {
  name = "subdomain.example.com"
}

resource aws_route53_record ns_record_for_subdomain {
  name    = aws_route53_zone.example_com.name
  zone_id = data.aws_route53_zone.example_com.id
  records = [
    aws_route53_zone.subdomain_example_com.name_servers[0],
    aws_route53_zone.subdomain_example_com.name_servers[1],
    aws_route53_zone.subdomain_example_com.name_servers[2],
    aws_route53_zone.subdomain_example_com.name_servers[3]
  ]
  ttl  = 300
  type = "NS"
}

resource aws_acm_certificate example_com {
  domain_name               = data.aws_route53_zone.example_com.name
  subject_alternative_names = [format("*.%s", data.aws_route53_zone.example_com.name)]
  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }
}

resource aws_route53_record certificate {
  for_each = {
    for dvo in aws_acm_certificate.example_com.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.example_com.id
  allow_overwrite = true
}

resource aws_acm_certificate_validation cert {
  certificate_arn         = aws_acm_certificate.example_com.arn
  validation_record_fqdns = [for record in aws_route53_record.certificate : record.fqdn]
}

resource "aws_route53_record" "subdomain_example_com" {
  zone_id = aws_route53_zone.subdomain_example_com.zone_id
  name    = aws_route53_zone.subdomain_example_com.name
  type    = "A"

  # 実際に通信をさばくALBの情報
  alias {
    name                   = aws_lb.xxx.dns_name
    zone_id                = aws_lb.xxx.zone_id
    evaluate_target_health = true
  }
}
```

# 参考
- Amazon Route 53
    - https://aws.amazon.com/jp/route53/
- AWS Certificate Manager
    - https://aws.amazon.com/jp/certificate-manager/
- route53_zone
    - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_zone
- route53_record
    - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record
- acm_certificate
    - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/acm_certificate
- format Function
    - https://www.terraform.io/docs/configuration/functions/format.html
- acm_certificate_validation
    - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/acm_certificate_validation
- SANsにワイルドカードが入ったACMのDNS認証なSSL証明書をTerraformで作るときのハマりどころ
    - https://fusagiko.hatenablog.jp/entry/2019/09/30/223700
- Route 53 に登録されているドメインのホストゾーンの置き換え
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/domain-replace-hosted-zone.html
- サブドメインのトラフィックのルーティング
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/dns-routing-traffic-for-subdomains.html
- ドメインのネームサーバーおよびグルーレコードの追加あるいは変更
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/domain-name-servers-glue-records.html
- サポートされる DNS レコードタイプ
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/ResourceRecordTypes.html
- エイリアスレコードと非エイリアスレコードの選択
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html
- 新しいドメインの DNS ルーティングの設定
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/dns-configuring-new-domain.html
- DNSを使用したドメイン所有権の検証
    - https://docs.aws.amazon.com/ja_jp/acm/latest/userguide/gs-acm-validate-dns.html