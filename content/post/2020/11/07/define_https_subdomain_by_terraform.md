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

<!--more-->


検証環境に利用したTerraformとAWS Providerのバージョンは以下となる。

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

- example.com というドメインをRoute53の登録済みドメインとして登録済み
- example.comまたは*.example.comというようなサブドメインをHTTPS化するSSL証明書を発行する
- https://subdomain.example.com というようなHTTPS化されたサブドメインを定義する。



# Route53でホストゾーンを定義する
まずドメインとサブドメインを別々のホストゾーンを定義する。

- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_zone

```hcl
data aws_route53_zone example_com {
  name = "example.com"
}

resource aws_route53_zone subdomain_example_com {
  name = "subdomain.example.com"
}
```

Route53でexample.comを登録済みドメインにしている場合、すでにホストゾーンが作成されているはずである。

- 新しいドメインの DNS ルーティングの設定
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/dns-configuring-new-domain.html

Terraform管理外のリソースとしてこれを`data`で取り込んでいる。ルートドメインをTerraformで定義しようとしないこと。

## ルートドメインのホストゾーンはTerraformで作成しないこと
AWS Provider 3.6.0系でもRoute53の登録済みドメインをTerraformで管理することはできない。
Terraformでルートドメインを定義した場合、定義したルートドメインのNSコードと登録済みドメインのNSコードが一致しなくなるので証明書のDNS認証などができなくなる。。

- ドメインのネームサーバーおよびグルーレコードの追加あるいは変更
    - https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/domain-name-servers-glue-records.html

Terraforrmでドメインとサブドメインに関連性をもたせるコマンドは次のとおり。
もしルートドメインのホストゾーンを削除してしまった場合は、再作成後、登録済みドメインのネームサーバを作成したホストゾーンのNSレコードの値に更新する必要がある。


```hcl
resource aws_route53_record example_com {
  name    = aws_route53_zone.example_com.name
  zone_id = data.aws_route53_zone.budougumi0617.id
  records = [aws_route53_zone.automata_budougumi0617.name_servers[0],
    aws_route53_zone.example_com.name_servers[1],
    aws_route53_zone.example_com.name_servers[2],
    aws_route53_zone.example_com.name_servers[3]
  ]
  ttl  = 300
  type = "NS"
}
```

# ACMでHTTPS化用のSSL証明書を作成する

- DNSを使用したドメイン所有権の検証
    - https://docs.aws.amazon.com/ja_jp/acm/latest/userguide/gs-acm-validate-dns.html

## ワイルドカード付きの証明書を発行する

- https://aws.amazon.com/jp/premiumsupport/knowledge-center/acm-certificate-renewal/

- SANsにワイルドカードが入ったACMのDNS認証なSSL証明書をTerraformで作るときのハマりどころ
    - https://fusagiko.hatenablog.jp/entry/2019/09/30/223700


```hcl
resource aws_acm_certificate example_com {
  domain_name               = data.aws_route53_zone.example_com.name
  subject_alternative_names = [format("*.%s", data.aws_route53_zone.example_com.name)]
  # subject_alternative_names = [aws_route53_zone.subdomain_example_com.name]
  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }
}
```


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

records新しいのが詳しいかなと思いました。！


```
// apply時にSSL証明書の検証が完了するまで待機する設定。
resource aws_acm_certificate_validation cert {
  certificate_arn         = aws_acm_certificate.example_com.arn
  validation_record_fqdns = [for record in aws_route53_record.certificate : record.fqdn]
}
```

## ACM認証が終わらない