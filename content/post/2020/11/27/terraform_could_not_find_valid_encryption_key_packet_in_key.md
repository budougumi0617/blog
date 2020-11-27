+++
title= "TerraformレジストリにリリースしようとGPG Keyをimportしようとすると「Could not find valid encryption key packet in key ~~」とエラーになる"
date= 2020-11-27T09:44:44+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["terraform"]
tags = ["terraform","gpg","cd"]
keywords = ["Terraform","Terraform Registry"]
twitterImage = "logos/terraform.png"
+++

Terraform Registryへ自作Providerをリリースしようとしたときに半日ハマったのでメモ。

<!--more-->

# TL;DR
- Terrafomr Providerを自作したらRegistryに簡単に公開できる
    - https://www.terraform.io/docs/registry/providers/publishing.html
- GitHub Actionsから自動リリースしようとしたら次のエラーでハマった
    - `Could not find valid encryption key packet in key`
- version 2.1.17以上の`gpg`コマンドを使ってGPGキーを生成するときは次のコマンドで生成すること
    - `gpg --full-generate-key`

なお、本記事で利用している各種ツールのバージョンは次のとおり。

```bash
$ gpg --version
gpg (GnuPG) 2.2.24
libgcrypt 1.8.7
```

実行環境はmacOS Mojave 10.14.6になる。

# Terrafomr Providerをリリースする
任意のリソースがTerraformに対応していないとき、Terraform Providerを自作することでそのリソースをTerraformで管理できるようになる。

Terraform Providerの作りかたは簡単で、ホビーレベルならば次のチュートリアルを読むだけで開発可能だ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://learn.hashicorp.com/collections/terraform/providers" data-iframely-url="//cdn.iframe.ly/lHR2l7J?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

自作したProviderはTerraform Registryへ公開することで簡単に他の人と共有することができる。
リリース方法も丁寧なドキュメントがある。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.terraform.io/docs/registry/providers/publishing.html" data-iframely-url="//cdn.iframe.ly/BY3P7l4"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>


<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/hashicorp/terraform-provider-scaffolding" data-iframely-url="//cdn.iframe.ly/gaft2lJ"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

文中に紹介されている次の2ファイルをコピペするだけで、tagを打てばリリースされる自動デプロイ環境を構築できる。

- GitHubリポジトリにある`.goreleaser.yml`ファイル
- GitHub Actionsの定義（`release.yml`）

リリースにはGPGキーを使った署名が必要なのだが、ここでハマった。

# Could not find valid encryption key packet in key ~~
（後述のとおり、手順通りにやっていなかったのだが、）上記の手順通りにGPGキーを生成し、GitHub Actionsの設定をして動かした結果、次のエラーがでた。

https://github.com/budougumi0617/terraform-provider-pixela/runs/1440615508?check_suite_focus=true

```
Run paultyng/ghaction-import-gpg@v2.1.0
  with:
    git_user_signingkey: false
    git_commit_gpgsign: false
    git_tag_gpgsign: false
    git_push_gpgsign: false
  env:
    GOROOT: /opt/hostedtoolcache/go/1.14.12/x64
    GPG_PRIVATE_KEY: ***
  
    PASSPHRASE: ***
📣 GnuPG info
Version    : 2.2.4 (libgcrypt 1.8.1)
Libdir     : /usr/lib/x86_64-linux-gnu/gnupg
Libexecdir : /usr/lib/gnupg
Datadir    : /usr/share/gnupg
Homedir    : /home/runner/.gnupg
🔮 Checking GPG private key
Error: Could not find valid encryption key packet in key 6d50fd48c3b3d7ea
```

# gpgコマンドのバージョンが2.1.17以上ならばgpg --full-generate-keyでgpgキーを生成する
今回GPGキーを新しく生成していた。生成方法は上記公式ガイドにリンクされていたGitHubのページの手順を使った。

https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/generating-a-new-gpg-key

ここで次のように手順が分岐している箇所がある。

> Generate a GPG key pair. Since there are multiple versions of GPG, you may need to consult the relevant man page to find the appropriate key generation command. Your key must use RSA.
> 
> If you are on version 2.1.17 or greater, paste the text below to generate a GPG key pair.
> 
>   $ gpg --full-generate-key
> 
> If you are not on version 2.1.17 or greater, the gpg --full-generate-key command doesn't work. Paste the text below and skip to step 6.
> 
>   $ gpg --default-new-key-algo rsa4096 --gen-key

私は`gpg`コマンドのバージョンが`2.2.24`なのに横着して`gpg --default-new-key-algo rsa4096 --gen-key`コマンドでGPGキーを生成していた。結論としてはこれが間違っていた。  
`gpg --full-generate-key`コマンドでGPGキーを再生成したところ、明らかにエクスポート時のASCII文字列長が代わって、暗号鍵の検出が成功するようになった。


# 終わりに
今回の失敗は手順のななめ読みをせず、ちゃんと読んで実行しようねという例だった。  
非可逆な操作もあったりするので、業務で同じようなことをしないように気をつけたい…


## 余談
今週は自社ブログに寄稿もしているのでちょっと小ぶりの更新。  
この記事はTerraform Providerを作る人にしか役に立たないが、寄稿記事はいろいろなシーンで使えるのでオススメ。

- GitHub Actionsとrelease-it npmでリリース作業を自動化する
    - https://devblog.thebase.in/entry/automatic-release-on-github-actions

# 参考
- https://learn.hashicorp.com/collections/terraform/providers
- https://www.terraform.io/docs/registry/providers/publishing.html
- https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/generating-a-new-gpg-key
- https://github.com/hashicorp/terraform-provider-scaffolding