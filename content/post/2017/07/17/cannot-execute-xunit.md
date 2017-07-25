+++
title= "VS for MacのテストエクスプローラーからXUnit(.NET Core1.1)が実行できない"
date= 2017-07-17T23:41:03+09:00
draft = false
slug = ""
categories = ["C#", "dot-net"]
tags = ["VS4Mac", "XUnit"]
author = "budougumi0617"
+++

# TL;DR
Visual Studio for Macから`XUnit`プロジェクトのテストが実行できないときは、コンソールから`dotnet test`コマンドを実行してみて出力を確認してみます。


# XUnit(.NET Core1.1)が実行できない

Visual Studio for Macの「単体テスト」ウインドウの「テストの実行」操作から、別PCで作成した.NET Core1.1`プロジェクトのXUnitを動かそうとしたのですが、実行が終わらない、「テスト結果」ウインドウの「出力」にも何も表示されない状態になりました。

# 解決方法

2017/07/17時点のVisual Studio for Macは、.NET Core1.1`形式のXUnitプロジェクトの実行に`dotnet test`コマンドを利用しています。ターミナルから左記のコマンドを実行することで、Visual Studio for Macのバックグラウンドで何が起きているか、原因を探ることができます。

[dotnet-test](https://docs.microsoft.com/ja-jp/dotnet/core/tools/dotnet-test)

コマンドを実行した結果、私の場合は、`.NET Core1.1.2`がインストールされていないことが原因でした(`.NET Core1.1.1`だと動かなかった)。

```shell
$ dotnet test TestProject/TestProject.csproj

...

The specified framework 'Microsoft.NETCore.App', version '1.1.2' was not found.
  - Check application dependencies and target a framework version installed at:
      /opt/dotnet/shared/Microsoft.NETCore.App
  - The following versions are installed:
      1.1.1
  - Alternatively, install the framework version '1.1.2'.
```

`.NET Core`は[2.0のプレビュー版がすでに公開](https://www.microsoft.com/net/core/preview#macos)されていたりして、近いうちにまた更新忘れのエラーに遭いそうな感じです。同じ目に合ったときのため、備忘録として書いておきます。

# 参考文献
[dotnet-test](https://docs.microsoft.com/ja-jp/dotnet/core/tools/dotnet-test)

[Download .NET Core](https://www.microsoft.com/net/download/core)

[Install for macOS 10.12 or higher (64 bit)](https://www.microsoft.com/net/core/preview#macos)
