+++
title = "[発表資料]Spinnakerを利用した自動カナリー分析 #WEBエンジニア勉強会08"
date = 2018-07-27T14:07:39+09:00
draft = false
toc = true
comments = true
author = "budougumi0617"
categories = ["presentation", "spinnaker"]
tags = ["spinnaker", "kayenta"]
+++


WEBエンジニア勉強会 #08の発表資料と資料中の参考リンク、補足をまとめた。

<!--more-->

|イベント名|WEBエンジニア勉強会 #08|
|---|---|
|URL|https://web-engineer-meetup.connpass.com/event/94200/|
|会場|アクアミーティングスペース渋谷|
|日時|2018/07/27(金)19:30 〜 21:30|
|ハッシュタグ| [#WEBエンジニア勉強会08](https://twitter.com/hashtag/WEB%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%8B%E3%82%A2%E5%8B%89%E5%BC%B7%E4%BC%9A08)|


---

- https://speakerdeck.com/budougumi0617/automated-canary-analysis-by-spinnaker-with-kayenta

<script async class="speakerdeck-embed" data-id="06d9e5ce9eb345cea57e6b81ff54537b" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# 継続的デリバリー
- https://en.wikipedia.org/wiki/Continuous_delivery

日本語のWikipediaにはまだない（継続的インテグレーションはある）

同タイトル本は表紙の「いつまで手動でデプロイしているんですか」という帯が挑戦的

- 継続的デリバリー
  - http://amazon.jp/dp/4048930583

# カナリーリリース
- https://makitani.net/shimauma/canary-release

カナリアリリースともいう。
ガス漏れを検知するために鳥のカナリアを炭鉱に持ち込んで「確認」したことに由来している

# Spinnaker
- https://www.spinnaker.io/

継続的デリバリーを行なうためのOSS。今回紹介した機能の他にもカオスモンキーテストなども行えるとのこと。

# Kayenta
- https://github.com/spinnaker/kayenta

自動カナリー分析を行なうためのSpinnakerのサブモジュール。詳細は以下の記事などを参照のこと

- Automated Canary Analysis at Netflix with Kayenta
  - https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69

- Kayenta : Google と Netflix が共同開発したオープンソースの自動カナリア分析ツール
  - https://cloudplatform-jp.googleblog.com/2018/06/introducing-Kayenta-an-open-automated-canary-analysis-tool-from-Google-and-Netflix.html

# デモについて
- AUTOMATED CANARY ANALYSIS USING SPINNAKER - CODELAB
  - https://ordina-jworks.github.io/cloud/2018/06/01/Automated-Canary-Analysis-using-Spinnaker.html
  
デモで作ったパイプラインなどは上記の記事を参考に構築されている。また、ブログでも以前に一度まとめている。

- [Spinnakerを使ったカナリーリリースの自動評価 #spinnaker #kayenta](/2018/06/29/get-started-auto-canary-analysis-by-spinnaker-with-kayenta/)

# 関連
- [Spinnakerを使ったカナリーリリースの自動評価 #spinnaker #kayenta](/2018/06/29/get-started-auto-canary-analysis-by-spinnaker-with-kayenta/)
- [Continuous Delivery With Spinnakerでマイクロサービスの継続的デリバリーの課題と解決手法を学ぶ](/2018/06/14/review-continuous-delivery-with-spinnaker/)
- [Spinnakerの関連記事](/categories/spinnaker/)
