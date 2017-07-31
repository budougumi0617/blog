+++
title= ".NET Standard1.6プロジェクトのCI環境を構築する。"
date= 2017-07-25T20:46:26+09:00
draft = false
slug = ""
categories = ["dot-net", "c-sharp", "DevOps"]
tags = ["AppVeyor", "Coveralls", "OpenCover", "XUnit"]
author = "budougumi0617"
+++

# TL;DR

`.NET Standard1.6`ベースで`Nuget`パッケージを作っています。開発で利用しているCI環境の構築方法です。実際に利用しているリポジトリは以下になります。

[![Build status](https://ci.appveyor.com/api/projects/status/nv8feqr5attxrx5j?svg=true)](https://ci.appveyor.com/project/budougumi0617/testable)
[![codecov](https://codecov.io/gh/budougumi0617/Testable/branch/master/graph/badge.svg)](https://codecov.io/gh/budougumi0617/Testable)

[https://github.com/budougumi0617/Testable](https://github.com/budougumi0617/Testable)


現状出来ているのは以下です。`git tag`で自動パッケージリリースなども追って対応していきたいなと思っています。


- `GitHub`のリポジトリが更新されたとき、自動でビルドが実施される。
- ビルド時にリポジトリに同梱されているテストプロジェクトが自動で実行される。
- テストのカバレッジを計測し、`Codecov`で結果を可視化する。
- バッジでビルド結果とカバレッジ率がわかる。

`.NET Standard1.6(VS2017)`環境で構築したプロジェクト向けの記事なので、その他の環境の場合はまず公式サイトなどを参考にしてください。

[CodeCov Test Coverage Integration](https://www.appveyor.com/blog/2017/03/17/codecov/)

[AppVeyorとCodecovを使ってC#のコードカバレッジを計測する](https://yoshinorin.net/2016/09/21/appveyor-codecov-csharp/)

# 構築手順

## 利用ツール/サービス
基本的に無料のツールで構築していきます。

|用途|ツール名|
|---|---|
|IDE|`Visual Studio2017`/`Visual Studio for Mac`|
|テスト|`XUnit`/`OpenCover`|
|構成管理|`GitHub`|
|CI|`AppVeyor`/`Coveralls`|

以降では、GitHubリポジトリに`.NET1.6`プロジェクトが入った状態のあとの説明です。

## `XUnit`プロジェクトを作成する。
まず、以下の記事を参考に、テストプロジェクトを作成してください。

[.NET StandardプロジェクトをxUnitでテストする方法](http://www.nuits.jp/entry/2017/06/11/132339)

`VS for Mac`の場合は、「新しいプロジェクトの追加」から、「`xUnit Test Project`」を選択することで同等のプロジェクトを作成することができます。`.NET1.6`プロジェクトを参照するテストプロジェクトを作成する場合は`.NET Core`プロジェクトで作るのが無難です。

その後、カバレッジ計測に必要な、`xunit.runner.console`と`OpenCover`も`Nuget`からテストプロジェクトに追加しておいてください。全てを行うと、テストプロジェクトの`.csproj`ファイルには以下のパッケージが追加されているはずです。

```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="15.0.0" />
    <PackageReference Include="xunit" Version="2.2.0" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.2.0" />
    <PackageReference Include="xunit.runner.console" Version="2.2.0" />
    <PackageReference Include="OpenCover" Version="4.6.519" />
  </ItemGroup>
```

## テスト対象のプロジェクトに`DebugType`属性を追加する。
`VS2017`以降で作成した`.NET Standard`プロジェクトのカバレッジを、`OpenCover`で測るためには、テスト対象のプロジェクトの`.csproj`ファイルの`PropertyGroup`に`<DebugType>full</DebugType>`を追加する必要があります。

```xml
  <PropertyGroup>
    <TargetFramework>netstandard1.6</TargetFramework>
    <DebugType>full</DebugType>
  </PropertyGroup>
  ```

詳細については、以下を参照してください。

[OpenCoverでVS2017でビルドした.NETプロジェクトのカバレッジを測る](https://budougumi0617.github.io/post/2017/07/13/opencover-to-vs2017/)


## `Codecov`の設定ファイルの作成

[Codecov](https://codecov.io/)

Codecovはカバレッジの結果を可視化してくれるオンラインサービスです。GitHubへの通知なども行えるので、例えば「PRのカバレッジが、コード変更前よりX%下がったとき失敗通知をGitHubへ送る」などと言った設定もできます。
リポジトリのルートディレクトリに`.codecov.yml`を配置しておくことで、諸々のしきい値などを設定できます。私はいつもテンプレを使っています。

[Testable/.codecov.yml](https://github.com/budougumi0617/Testable/blob/master/.codecov.yml)

各パラメータの詳細は以下のWikiを参照してください。

[Codecov Yaml](https://github.com/codecov/support/wiki/Codecov-Yaml)

## `Appveyor`の設定

[Appveyor](https://ci.appveyor.com/)

AppveyorはWindowsコンテナで実行できるCIツールです。
リポジトリのルートディレクトリに、以下の`appveyor.yml`ファイルを配置します。

[Testable/appveyor.yml](https://github.com/budougumi0617/Testable/blob/master/appveyor.yml)

```yml
version: 0.0.{build}
pull_requests:
  do_not_increment_build_number: true
image: Visual Studio 2017
configuration: Debug
platform: Any CPU
before_build:
- cmd: >-
    git submodule init
    git submodule update --init --recursive
    dotnet restore Testable.sln
build:
  project: Testable.sln
  verbosity: minimal

# Set OpenCover setting according to below URL
# https://github.com/OpenCover/opencover/wiki/Usage

test_script:
- ps: >-
    $opencover = (Resolve-Path "~\.nuget\packages\OpenCover\4.6.519\tools\OpenCover.Console.exe").ToString()

    $filter = "+[Testable*]* -[*.Tests*]*"

    regsvr32 x86\OpenCover.Profiler.dll

    regsvr32 x64\OpenCover.Profiler.dll

    & $opencover -target:"c:\Program Files\dotnet\dotnet.exe" `
    -targetargs:"test -f netcoreapp1.1 -c Debug Testable.Tests/Testable.Tests.csproj" `
    -mergeoutput `
    -hideskipped:File `
    -output:opencoverCoverage.xml `
    -oldStyle `
    -filter:$filter `
    -searchdirs:Testable.Tests/bin/Debug/netcoreapp1.1 `
    -register:user
    
    $env:Path = "C:\Python34;C:\Python34\Scripts;$env:Path"

    python -m pip install --upgrade pip

    pip install codecov

    &{codecov -f "opencoverCoverage.xml"}

notifications:
- provider: GitHubPullRequest
  on_build_success: true
  on_build_failure: true
  on_build_status_changed: true
```

主要なところを解説しておきます。`filter`やプロジェクト名の置き換えは[OpenCoverのWiki](https://github.com/OpenCover/opencover/wiki/Usage)を参考に行ってください。

|`yml`の記載|解説|
|---|---|
|`image: Visual Studio 2017`| `VS2017`が入っているイメージコンテナを使います。
|`dotnet restore Testable.sln`|Nugetの復元は`dotnet`コマンドで出来ます。
|`- ps: >-`|ここから下が`Powershell`スクリプトです。 **処理と処理の間に空行を入れないと正しく実行されないので注意してください。**
| `$runner = "dotnet test"`|テストも`dotnet`コマンドで実行します。
|`pip install codecov`|おまじないです。`Appveyor`/`Codecov`連携の詳細は[この辺](https://www.appveyor.com/blog/2017/03/17/codecov/)を参考に。



## バッジを貼る
あとは各種サービスにログインして、リポジトリに対してサービスをONにしてください。バッジの貼り方など、このへんは既存のやり方と変わらないので省略します。以下の記事などを参考にどうぞ。

[CodeCov Test Coverage Integration](https://www.appveyor.com/blog/2017/03/17/codecov/)
[AppVeyorとCodecovを使ってC#のコードカバレッジを計測する](https://yoshinorin.net/2016/09/21/appveyor-codecov-csharp/)


# おしまい
以上で設定はおしまいです。エラーが出ていないのにカバレッジが取れないときは、

- テスト対象のプロジェクトに`DebugType`属性をつけるのを忘れていないか
- `OpenCover`の`filter`の設定がおかしくなっていないか

などを確認してください。とくに`DebugType`属性が今までと違うところでハマりました。。。


---


### ツールなどの選定についての補足
各サービス、フレームワークを選んだ(選べなかった)理由を解説しておきます。

## `Appveyor` or `Visual Studio Team Services/Teeam Foundation Server`
`VSTS`で`GitHub`に貼れるテストカバレッジのバッチを作る方法がなかったので、`Appveyor`となりました。
`.NET Standard`なのでLinuxコンテナのCIサービスでもビルドは出来るのですが、`dotnet`コマンドなどが標準のコンテナで使える`Appveyor`を使います。

## `MSTest` or `XUnit` or `NUnit`
`MSTest`は`Visual Studio for Mac`では使えません。なので選べません。`NUnit`の場合、`.NET1.6`プロジェクトで利用するには`NUnit3`である必要があります。が、`OpenCover`が`NUnit3`をフルサポートしていないようなので`XUnit`にしました。

## `Coveralls`or `Codedev`
Coverallsですと`C#`向けにプラグインがあるので連携が簡単です。ただ、`Codecov`でも流用できるのと、UIが好みなので`Codecov`使ってます。
