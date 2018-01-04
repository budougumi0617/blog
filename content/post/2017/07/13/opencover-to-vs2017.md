+++
title= "OpenCoverでVS2017でビルドした.NETプロジェクトのカバレッジを測る"
date= 2017-07-13T09:25:36+09:00
draft = false
slug = ""
categories = ["C-sharp", "dot-net"]
tags = ["OpenOver", "DevOps", "VS2017"]
author = "budougumi0617"
aliases =  ["/post/2017/07/13/opencover-to-vs2017/"]
+++

# TL;DR
OpenCoverでVisual Studio2017でビルドした`.NET Core/Standard`のプロジェクトのコードカバレッジを計測したいときは、`.csproj`ファイルに`DebugType`を`full`で追加すること。

````xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard1.6</TargetFramework>
    <DebugType>full</DebugType>
  </PropertyGroup>

</Project>
````

# OpenCoverでカバレッジが計測できない。

Visual Studio2017から.NET Core, .NET Standardのプロジェクトの構成ファイル(`.csproj`)の形式が変更になっています。

[++C++; // 未確認飛行 C  - 新しい csproj 形式](http://ufcpp.net/blog/2017/5/newcsproj/)

これに合わせてか、ビルド時に生成されるPDBファイル(`.pdb`)の情報ファイルの内容も変更になっているようです。
そのため、通常従来の形式を期待して`.pdb`ファイルを解析する`OpenCover`をそのまま使ってもカバレッジは計測できません。

```
ProcessModel: Default    DomainUsage: Single
Execution Runtime: net-3.5
.Cannot instrument C:\projects\testable\Testable.Tests\bin\Debug\Testable.dll as no PDB/MDB could be loaded
.
Tests run: 2, Errors: 0, Failures: 0, Inconclusive: 0, Time: 1.4310438 seconds
  Not run: 0, Invalid: 0, Ignored: 0, Skipped: 0
Committing...

```

# 解決方法

これを解消するには、`.pdb`ファイルの情報を従来形式にしてビルドする必要があります。カバレッジを計測したいプロジェクトの`.csproj`ファイル内で該当属性を`full`にすることで`OpenCover`でもカバレッジを計測できるようになります。

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard1.6</TargetFramework>
    <DebugType Condition="'$(Configuration)'=='DEBUG'">full</DebugType>
  </PropertyGroup>

</Project>
```


上記例は[前述の未確認飛行さんの記事](http://ufcpp.net/blog/2017/5/newcsproj/)を参考に、Debugビルド時のみに制限した設定にしています。


# 参考文献
[++C++; // 未確認飛行 C  - 新しい csproj 形式](http://ufcpp.net/blog/2017/5/newcsproj/)

[Csc タスク](https://msdn.microsoft.com/ja-jp/library/s5c8athz.aspx)


