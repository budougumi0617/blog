+++
title = "[書評]Google Cloud Platform エンタープライズ設計ガイドを読んだ"
date = 2018-08-04T07:03:18+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["review", "gcp"]
tags = ["gcp", "book"]
+++

[Google Cloud Platform エンタープライズ設計ガイド](http://amazon.jp/dp/4822257908)を読んだので読書メモ

<!--more-->

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4822257908&linkId=623082059a17c3c39dffd10ca5b94856"></iframe>


# 所感
自分はGKEは少し触っているのだが、他のGCPのサービスはまだちゃんと使えていない。
GCPにどんなサービスがあるのか知るために表題の本を購入した。

## とりあえずGCPの主要機能をざっくり俯瞰したいときにおすすめ
本書では主要なGCPのサービスが項目別に各章にまとめられている。
第3章のストレージサービスの章ではサービス別の具体的な使用用途など説明されおり、類似サービスをどのように使い分ければよいか学ぶことができた。
また、10章、11章にはサンプルでオンプレ構成からGCP構成にリプレースするときの例が記載されている。
「機能はわかったけど、どう組み合わせるのだろう？」というところを知りたい人は構成例が参考になるだろう。

## 各サービスをどう操作するのかは書いていない
`gcloud`コマンドの説明や`Cloud Shell`についてはほぼまったく触れられていない。
付録の`Cloud Datalab`の使い方の中で登場するが、本章内では具体的な操作の例などは基本的に出てこない。
そのため、実際にGAEやGKEにデプロイをしてみたい人は別の本を探すか（といってもGCPの本はほとんどないが…）、
GCP公式の[チュートリアル](https://cloud.google.com/docs/)や[Google Codelabs](https://codelabs.developers.google.com/)で勉強したほうが良さそうだ。

- https://cloud.google.com/docs/
- https://codelabs.developers.google.com/

実際にGCP上でもろもろを構成する際はサービスアカウントの作成・設定が必要になったり、サービス間の権限設定をしないといけないと思うのだが、
そのへんの解説もとくにはない(IAM自体の紹介は7章に記載がある）。

# 終わりに
自分はあまり英語が得意ではないので、適度なボリュームの日本語でGCPの主要サービスの概要を知ることが出来てよかった。
AWSの類似サービスも適宜挙げられているので、AWSを一通り触っている人には一層のことわかりやすいと思う。
欲を言うと、個人的には「手の動かし方」を知りたかったので、もう少し具体的な操作などが書いてあると良かった。
自分と同じ内容を求める人にとっては、[プログラマのためのGoogle Cloud Platform入門](http://amazon.jp/dp/B0721JNVGT)のほうがよいかもしれない。こちらは少し古いせいかサンプルコードが動かないなどの問題があるらしい（自分は一度目を通しただけでまだサンプルコードなどは動かせていない）。


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4822257908&linkId=623082059a17c3c39dffd10ca5b94856"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B0721JNVGT&linkId=ac5fabb7a7cee775ede839a748df230d"></iframe>
