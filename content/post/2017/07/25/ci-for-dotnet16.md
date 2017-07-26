+++
title= ".NET Standard1.6プロジェクトのCI環境を構築する。"
date= 2017-07-25T20:46:26+09:00
draft = true
slug = ""
categories = ["dot-net", "C-sharp"]
tags = ["DevOps", "OpenCover", "XUnit"]
author = "budougumi0617"
+++


`.NET Standard1.6`ベースでNugetパッケージを作っています。開発で利用しているCI環境の構築方法です。

## 実現できること
- GitHubのコードが更新されたとき、自動でビルドが実施される
- リポジトリに同梱されているテストプロジェクトが自動で実行される
- テストのカバレッジを計測し、Coverallsに渡される。

