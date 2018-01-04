+++
title= "Xamarin.Macプロジェクトのオプションにある「追加のmmp引数」に指定できる値"
date= 2017-07-26T22:24:00+09:00
draft = false
slug = ""
categories = ["Xamarin.Mac", "c-sharp", "VisualStudio"]
tags = ["xamarin", "mmp"]
author = "budougumi0617"
aliases = ["/post/2017/07/26/xamarin-mmp/"]
+++

# TL;DR
`Visual Studio for Mac`で`Cocoa App`のプロジェクトをビルドするとき、「`追加のmmp引数`」を設定しておくと`.app`ファイルを作成するときに詳細なオプションが渡せます。`.csproj`ファイル内では`<MonoBundlingExtraArgs>`の属性です。指定できる引数は以下のコマンドで確認することが出来ます。

````sh
$ /Library/Frameworks/Xamarin.Mac.framework/Versions/Current/bin/mmp -?
Copyright 2010 Novell Inc.
Copyright 2011-2016 Xamarin Inc.
Usage: mmp [options] application-exe
  -h, -?, --help             Displays the help
      --version              Output version information and exit.
  -f, --force                Forces the recompilation of code, regardless of
  etc...
````

# `追加のmmp引数`という謎のオプション

 `Visual Studio for Mac`で`Cocoa App`プロジェクトを作成すると、プロジェクトのプロジェクトオプションに、「`Mac Build`」という項目があります。このオプション一覧には「`追加のmmp引数`」というテキストフィールドのオプションがあるのですが、バルーンヘルプを見ても「`アプリケーションバンドルツールmmpに渡される追加のコマンドライン引数`」としか出てきません。バリデーションもなく、基本なんでも入力できるので、正直何に使うのかよくわかっていませんでした。

Xamarinの公式ページなどでも少し触れられている程度です。

[Xamarin.Mac Internals#Enabling the Partial Static Registrar](https://developer.xamarin.com/guides/mac/advanced_topics/internals/#Enabling_the_Partial_Static_Registrar)

> The Partial Static Registrar is enabled in Xamarin.Mac by double-clicking the Project Name in the Solution Explorer, navigating to Mac Build and adding --registrar:static to the Additional mmp arguments: field. For example:

# `mmp` とは
`mmp` は`Xamarin.Mac`で作成された`C#`の一連のバイナリからMacのネイティブアプリ(`.app`)を作る過程で呼ばれているツールです（`Mac Build`オプション内で言及されているので、当然と言えば当然ですね…)。
その`mmp`コマンドに引数を渡して具体的に何が出来るかというと、例えばビルド中の特定の警告を抑制(`--nowarn:${WARN_NUM}`)したり、あるいは特定の警告をビルドエラー(`--warnaserror:${WARN_NUM}`)扱いに出来ます。`.csproj`ファイルで言うと、`<MonoBundlingExtraArgs>`属性になります。以下のような`.csproj`ファイルの内容の場合、`Debug|x64`ビルド時に、警告`MM0617`の抑制をオプション指定したことになります。

```xml
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|x64' ">
    ...
    <MonoBundlingExtraArgs>--nowarn:0617</MonoBundlingExtraArgs>
    ...
  </PropertyGroup>
```

# `mmp`にオプション引数として指定できる値の一覧

では、どのようなオプション値が指定できるのかというと、あまりネット上には情報がありませんでした。ただ、`mmp`コマンド自体は`-h`コマンドオプションが実装されているので、ローカルの`mmp`コマンドを直接実行してみることで、渡せるコマンドオプションがわかります。私のMacでは`mmp`コマンドにパスが通ってなかったので、以下のようにしてオプション一覧を確認しました。各オプションの値もある程度解説されています。

```
/Library/Frameworks/Xamarin.Mac.framework/Versions/Current/bin/mmp -?
mmp - Xamarin.Mac Packer
Copyright 2010 Novell Inc.
Copyright 2011-2016 Xamarin Inc.
Usage: mmp [options] application-exe
  -h, -?, --help             Displays the help
      --version              Output version information and exit.
  -f, --force                Forces the recompilation of code,
                             regardless of timestamps
      --cache=VALUE          Specify the directory where temporary build files will be cached
  -a, --assembly=VALUE       Add an assembly to be processed
  -r, --resource=VALUE       Add a resource to be included
  -o, --output=VALUE         Specify the output path
  -n, --name=VALUE           Specify the application name
  -d, --debug                Build a debug bundle
      --nolink               Do not link the assemblies
      --mapinject            Inject a fast method map [deprecated]
      --minos=VALUE          Minimum supported version of Mac OS X
      --linksdkonly          Link only the SDK assemblies
      --linkskip=VALUE       Skip linking of the specified assembly
      --i18n=VALUE           List of i18n assemblies to copy to the output
                               directory, separated by commas (none,all,cjk,
                               mideast,other,rare,west)
    ...
    省略
    ...

      --warnaserror[=VALUE]  An optional comma-separated list of warning codes
                               that should be reported as errors (if no
                               warnings are specified all warnings are reported
                               as errors).
      --nowarn[=VALUE]       An optional comma-separated list of warning codes
                               to ignore (if no warnings are specified all
                               warnings are ignored).
    etc...
```
