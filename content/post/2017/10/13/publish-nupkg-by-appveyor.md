+++
title= "GitHubのタグに連動して、.NETStandard1.6のNugetパッケージを自動リリースする。"
date= 2017-10-13T17:06:29+09:00
draft = false
slug = ""
categories = ["dot-net", "c-sharp", "DevOps"]
tags = ["Appveyor", "nuget.org", "devops"]
author = "budougumi0617"
aliases = ["/post/2017/10/13/publish-nupkg-by-appveyor/"]
+++

# TL;DR
`.NET Standard1.6`ベースでNugetパッケージを作っています。`GitHub`でリリースしたとき(`git tag`をプッシュしたとき)、自動的に[nuget.org](https://www.nuget.org/.)上のパッケージが更新されるようにしました。

https://ci.appveyor.com/project/budougumi0617/testable/build/0.0.74

```
---------------------------PRや通常のプッシュをトリガーにビルドしたとき
Uploading artifacts...
[1/1] Budougumi0617.Testable.0.0.5.nupkg (5,100 bytes)...100%
"NuGet" deployment has been skipped as environment variable has not matched ("appveyor_repo_tag" is "false", should be "true")
Build success

---------------------------タグをトリガーにビルドしたとき
Uploading artifacts...
[1/1] Budougumi0617.Testable.0.0.5.nupkg (5,101 bytes)...100%
Deploying using NuGet provider
Publishing Budougumi0617.Testable.0.0.5.nupkg to https://www.nuget.org/api/v2/package...OK
Total packages published: 1
Build success
```

実際に利用しているリポジトリは以下になります。

### budougumi0617.Testable
[![NuGet version](https://badge.fury.io/nu/budougumi0617.Testable.svg)](https://badge.fury.io/nu/budougumi0617.Testable)
[![Build status](https://ci.appveyor.com/api/projects/status/nv8feqr5attxrx5j?svg=true)](https://ci.appveyor.com/project/budougumi0617/testable)
[![codecov](https://codecov.io/gh/budougumi0617/Testable/branch/master/graph/badge.svg)](https://codecov.io/gh/budougumi0617/Testable)

https://github.com/budougumi0617/Testable

# 関連ツールや対象プロジェクトについて

|用途|ツール名|
|---|---|
|構成管理|`GitHub`|
|自動ビルド/自動デプロイ|`AppVeyor`|
|リリース対象の`nupkg`|`.NET Standard1.6`プロジェクト|


# 構築手順

継続的インテグレーション（自動ビルド、自動テスト）までは別途構築済みとします。
詳細は以下を参考にしてください。

[.NET Standard1.6プロジェクトのCI環境を構築する。](/post/2017/07/25/ci-for-dotnet16/)

# リリースに関する設定
AppveyorはWindowsコンテナで実行できるCIツールです。 
`.NET`系の開発で使うツールや設定はプリセッティング/標準サポートされています。

Nugetの公開も基本的に公式の手順の通りにやればOKです。

[AppVeyor | Publishing to NuGet feed](https://www.appveyor.com/docs/deployment/nuget/)

リポジトリのルートディレクトリに、`appveyor.yml`ファイルを配置します。
以下はリリースに関わる関連設定の抜粋です。

[Testable/appveyor.yml](https://github.com/budougumi0617/Testable/blob/master/appveyor.yml)

```yml
image: Visual Studio 2017
configuration: Debug
# platform: Any CPU
build:
  publish_nuget: true
  project: Testable.sln
  verbosity: minimal

deploy:
- provider: NuGet
  api_key:
    # Use encrypt tool
    # https://ci.appveyor.com/tools/encrypt
    secure: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
  artifact: /.*\.nupkg/
  on:
    appveyor_repo_tag: true
```

.NET Standard1.6プロジェクトの場合はビルドにVisual Studio2017を使う必要があります。`platform: Any CPU`を設定すると、成果物のパスの解決に失敗してしまうため、`configuration`のみ設定しています。

## リリースする`nupkg`ファイルをビルドする。
ビルドの設定の中で`publish_nuget: true`としておくと、自動的に`nuget.org`にアップロードする`nupkg`ファイルが生成されます。

## リリース時の認証に使うAPIキーを設定する。
`nuget.org`にアカウントを作成済みならば、以下のページからAPIキーを作成できます。

[Nuget Gallery| API Keys](https://www.nuget.org/account/ApiKeys)

リポジトリの`yml`に直接記載する場合は、Appveyor提供の暗号化ツールで暗号化してから使います。

[AppVeyor | Encrypt configuration data](https://ci.appveyor.com/tools/encrypt)

AppVeyorのGUI画面から環境変数として登録しておき、、`env:NUGET_KEY`などのように呼んでも良いです。

## `tag`がプッシュされたときだけリリースする。
`tag`がプッシュされたときだけ`deploy`を実行するには`appveyor_repo_tag: true`を設定します。公式の説明は以下です。

[AppVeyor | Build on tags (GitHub, GitLab and BitBucket only)](https://www.appveyor.com/docs/branches/#build-on-tags-github-gitlab-and-bitbucket-only)

以上で`GitHub`にタグがプッシュされたときだけ、`nuget.org`にパッケージがリリースされるようになりました。

https://ci.appveyor.com/project/budougumi0617/testable/build/0.0.74

```
---------------------------PRや通常のプッシュをトリガーにビルドしたとき
Uploading artifacts...
[1/1] Budougumi0617.Testable.0.0.5.nupkg (5,100 bytes)...100%
"NuGet" deployment has been skipped as environment variable has not matched ("appveyor_repo_tag" is "false", should be "true")
Build success

---------------------------タグをトリガーにビルドしたとき
Uploading artifacts...
[1/1] Budougumi0617.Testable.0.0.5.nupkg (5,101 bytes)...100%
Deploying using NuGet provider
Publishing Budougumi0617.Testable.0.0.5.nupkg to https://www.nuget.org/api/v2/package...OK
Total packages published: 1
Build success
```


# 参考文献

[2017年 .NET Core 1.x (Visual Studio 2017) プロジェクト向け AppVeyor での自動ビルドの設定](http://tech.tanaka733.net/entry/cicd-in-appveyor-for-dotnetcore1x-2017)

[AppVeyor | Build pipeline](https://www.appveyor.com/docs/build-configuration/#build-pipeline)

[AppVeyor | Publishing to NuGet feed](https://www.appveyor.com/docs/deployment/nuget/)

[AppVeyor | Build on tags (GitHub, GitLab and BitBucket only)](https://www.appveyor.com/docs/branches/#build-on-tags-github-gitlab-and-bitbucket-only)
