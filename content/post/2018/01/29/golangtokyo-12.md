+++
title= "[Go]golang.tokyo #12 参加メモ #golangtokyo"
date= 2018-01-29T19:29:08+09:00
draft = false
slug = ""
categories = ["report","go"]
tags = ["golang","golangtokyo","k8s"]
author = "budougumi0617"
+++


golang.tokyo #12
テーマ：『スタートアップ・新規事業におけるGo』


|||
|---|---|
|URL|https://golangtokyo.connpass.com/event/76540/|
|会場|株式会社ネクストカレンシー（DMMグループセミナールーム）|
|日時|2018/01/29(月) 19:10 〜 22:00|
|Toggeter| https://togetter.com/li/1194466 |


# Next Currency とGAE/Go (仮)
sonatard 株式会社ネクストカレンシー


# GoとGCPとkubernetesを使ったMF KESSAIの歴史
shinofara MF KESSAI株式会社


# AWS Lambda Go First Impression
https://speakerdeck.com/timakin/aws-lambda-go-first-impression  
[@timakin](https://twitter.com/__timakin__)

- AWS公式サポートのaws-lambda-go
  - https://github.com/aws/aws-lambda-go
- 今までGoをAWS Lambdaで使うときはApexを経由して実行していた。
  - http://apex.run/
- 構造的にほとんど違いがないので、Apexからaws-lambda-goへの移行はラク
- AWS曰く、積極的にグローバル変数で初期化をしようと言っている。
  - [Using Global State to Maximize Performance](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/go-programming-model-handler-types.html#go-programming-model-handler-execution-environment-reuse)
- ContextのなかのContext（GAEなども似たようなアプローチをしている）  
https://github.com/aws/aws-lambda-go/blob/28a6e9fa1fc02d45d9a8c0a8d22c7d5c3580aa03/lambdacontext/context.go#L80-L89
<script src="http://gist-it.appspot.com/https://github.com/aws/aws-lambda-go/blob/28a6e9fa1fc02d45d9a8c0a8d22c7d5c3580aa03/lambdacontext/context.go?slice=79:89"></script>
- vaidateArgmentsという情熱的なリフレクション  
https://github.com/aws/aws-lambda-go/blob/6e2e37798efbb1dfd8e9c6681702e683a6046517/lambda/handler.go#L37-L51
<script src="http://gist-it.appspot.com/https://github.com/aws/aws-lambda-go/blob/6e2e37798efbb1dfd8e9c6681702e683a6046517/lambda/handler.go?slice=36:51"></script>
- aws-lambda-goの実装コードは忍術みたいな実装をしていて、読むと勉強になる



