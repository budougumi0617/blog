+++
title= "[Xamarin.Mac]NStextFieldにテキストが入力されるたびに変更を検出する"
date= 2017-11-24T09:25:47+09:00
draft = false
slug = ""
categories = ["Xamarin.Mac"]
tags = ["xamarin", "NSTextField"]
author = "budougumi0617"
+++

知っているとすぐ出来るけど、知らないとちょっと時間がかかるシリーズ。

# TL;DR
- `Xamarin.Mac`で`C#`側で文字入力のイベントを検出する
- その1。`NSTextField.value`への`string`のプロパティをデータバインドする
- その2。`NSTextFieldDelegate`を実装したクラスを`NSTextField.Delegate`に紐付けておく


# `NStextField`や`NSSecureTextField`の変更を検知したい。

`Xamarin.Mac`で`NSTextField`や`NSSecureTextField`などに対する文字列入力イベントを検知し、処理を行う。その1のほうが直感的だしシンプル。

## その1。`NSTextField.value`への`string`のプロパティをデータバインドする

普通に `NSTextField`の`value`に`string`のプロパティをデータバインドするだけでOKだった。データバインドのやり方は公式ページを参照のこと。

[Data Binding and Key-Value Coding|Simple Data Binding](https://developer.xamarin.com/guides/mac/application_fundamentals/databinding/#Simple_Data_Binding)

`ViewController`に`string`のプロパティを作る。

```csharp
// ViewController.cs
    string _password = "";
    [Export("Password")]
    public string Password
    {
        get { return _password; }
        set
        {
            WillChangeValue(nameof(Password));
            Console.WriteLine("Changed.");
            _password = value;
            DidChangeValue(nameof(Password));
        }
    }
```


`XCode`で対象の`NSTextField`の`Bindings Inspector`でプロパティを紐付ける。

![XCode見たNSTextFieldのBindings Inspector](/2017/11/data-bind.png)

文字が一文字編集されるたびにプロパティの`Setter`が呼ばれる。

## その2。`NSTextFieldDelegate`を実装したクラスを`NSTextField.Delegate`に紐付けておく

基本的には通常の`Cocoa App`プログラミングと同様のことを`C#`で実施するだけ。

[NSTextFieldで入力を検出する](https://blog.piyo.tech/posts/2015-07-30-080500)



事前準備として、文字入力を検出したい`NSTextField`などのプロパティを`XCode`から`C#`のクラスへ自動生成しておく。
方法はXamarin公式ページの以下のセクションから「Synchronizing Changes with Xcode」までを行うことで作成できる。

[Hello, Mac|Outlets and Actions](https://developer.xamarin.com/guides/mac/getting_started/hello,_mac/#Outlets_and_Actions)


日本語で読みたい方はこちら

[Xamarin.Macを初めて触る人に贈る、Xamarin公式 Hello, Mac の日本語訳記事](https://qiita.com/toshi0607/items/34b3e04f9ad24a5d0153#outlet%E3%81%A8action)


```csharp
// ViewController.designer.cs
[Register("ViewController")]
partial class ViewController
{
    [Outlet]
    AppKit.NSTextField TextField { get; set; }
}
```

検出時に行いたい処理を`NSTextFieldDelegate`継承クラスに実装する。

https://developer.xamarin.com/api/type/MonoMac.AppKit.NSTextFieldDelegate/

```csharp
public class MyDeleGate : AppKit.NSTextFieldDelegate{
    public override void Changed(NSNotification notification)
    {
        // base.Changed(notification); は呼んではいけない。
        Console.WriteLine("Changed.");
    }
}
```

あとはこの`Delegate`インスタンスを`NSTextFieldなどに紐付けて終わり。

```csharp
// ViewController.cs
public partial class ViewController : AppKit.NSViewController
{        
    public override void ViewDidAppear()
    {
        TextField.Delegate = new MyDeleGate();
    }
}
```

---

そこそこ調べてまずその2の方法を見つけた。そのあとその1でも出来ることに気づいたのでかなり遠回りしてしまった。。。