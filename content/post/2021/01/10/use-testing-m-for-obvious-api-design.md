+++
title= "Use Testing M for Obvious Api Design"
date= 2021-01-09T22:55:02+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang","design"]
keywords = ["go","APIデザイン","test","設計"]
twitterImage = "logos/Go-Logo_Aqua.png"

+++

<!--more-->


# 共通ライブラリを作っている
業務で複数のGoのサービスを扱うようになり、loggingやhttp middlewareといった汎用性の高いコードを共通ライブラリとして抽出している。
[go-kit](https://github.com/go-kit/kit)のようなOSSとして公開されているものに頼るのもよい。
しかし特有のデータ構造だったり、利用している技術セットに最適化すると自作することになるのが多いのではないか。

# 何も知らず実装できるAPI設計
さまざまなサービスの微妙にことなるユースケース、Goの経験値が均一でない開発チームで誤使用なくライブラリを使ってもらうには明瞭なAPIデザインが求められる。
とくに背景を共有しなくてもこちらが想定する使い方をしてもらえるのが望ましい。
APIデザインの極意ではこのようなことを「無知モードでも利用できること」と書いてあります。

> それらのAPIが適切に作成されていれば、組み立ては簡単な作業になり、デバッグしたり、ソースコードを読んだり、パッチを適用したり、つまり、一般的に言えば、他の人が行った内容を理解することに時間を費やすことがなくなります。要するに、「無知モード」で行うことができます
> 
> Jaroslav Tulach. APIデザインの極意 Java/NetBeansアーキテクト探究ノート (Japanese Edition) (Kindle の位置No.1447-1449). Kindle 版. 



# m *testing.Mを使う
