+++
title= "Xmarin.Macアプリでネイティブメソッドの動的ロードを含むビルドを行う"
date= 2017-08-04T08:59:08+09:00
draft = false
slug = ""
categories = ["Xamarin.Mac", "c-sharp", "VisualStudio"]
tags = ["xamarin", "mmp","Visual Studio for Mac"]
author = "budougumi0617"
aliases = ["/post/2017/08/04/xamarin-link-flags/"]
+++

ネイティブライブラリの動的ロードを含む`Xamarin.Mac`アプリを作ろうとしたら、ビルドエラーに遭遇しました。

# TL;DR
- `MMP : error MM5109: Native linking failed with error code 1`というビルドエラーを解決したい。
- プロジェクトオプション-「Mac Build」タブ-「追加のmmp引数」に「`--link_flags="-Wl,-undefined,dynamic_lookup"`」と設定する。
- `Xamarin.Mac`アプリのプロジェクトに、動的ロードで解決するメソッドが含まれていてもビルド出来るようになる。

# `MMP : error MM5109: Native linking failed with error code 1`
`Xamarin.Mac`プロジェクトをビルドするとき、`C#`のアプリをネイティブアプリに変換する処理が走ります。このとき、`C#`の中でネイティブライブラリを動的にロードしているようなプロジェクトは以下のようなエラーでビルドが止まることがあります。`_C_FooFunction`、`_C_BarFunction`は動的にロードするライブラリの中で宣言されているメソッドです。

```
Target _CompileToNative:

...

    xcrun -sdk macosx clang -g -mmacosx-version-min=10.12 -arch x86_64 -fobjc-runtime=macosx -ObjC -u _C_FooFuncion -u _C_BarFunction  ....
    Undefined symbols for architecture x86_64:
      "_C_FooFunction", referenced from:
         -u command line option
      "_C_BarFunction", referenced from:
         -u command line option

...

    ld: symbol(s) not found for architecture x86_64
    clang : error : linker command failed with exit code 1 (use -v to see invocation)
    
    MMP : error MM5109: Native linking failed with error code 1.  Check build log for details.
Done building target "_CompileToNative" in project "CocoaApp.csproj" -- FAILED.
```

# `mmp`の`--link_flags`経由でリンカーにオプションを渡す

ネイティブアプリにするビルドの過程で`clang`が呼ばれ、そのなかでリンカー(`ld`)がシンボルを解決出来ないようでした。ビルド過程の`ld`コマンドにオプションを渡して、シンボルが未解決でもビルドを完了させるようにします。

- VSforMacでプロジェクトの「オプション」ウインドウを開く
- 「Mac Build」タブを選択する。
- 「追加のmmp引数」という入力欄に、「`--link_flags="-Wl,-undefined,dynamic_lookup"`」を追記する。

![プロジェクトオプション](/2017/08/link_flags.png)

これで未定義のメソッドが合っても、ビルドエラーが発生しなくなります。
当然実行時エラーが発生する恐れがあるので、ビルドしたアプリが正しく動くかは別途確認が必要です。`mmp`に渡せるオプションは他にもいくつかあるのですが、Web上にあまり情報がありません。オプションの調べ方は別記事にまとめました。

[Xamarin.Macプロジェクトのオプションにある「追加のmmp引数」に指定できる値](/post/2017/07/26/xamarin-mmp/)

ちなみに、「追加のmmp引数」近くに「リンカーの動作」というオプションもあるのですが、これでは解決しませんでした。


# 参考

[Error when making dynamic lib from .o](https://stackoverflow.com/questions/14090257/error-when-making-dynamic-lib-from-o)

[Xamarin.Macプロジェクトのオプションにある「追加のmmp引数」に指定できる値](/post/2017/07/26/xamarin-mmp/)
