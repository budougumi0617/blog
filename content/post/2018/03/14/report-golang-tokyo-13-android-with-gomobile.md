+++
title= "[Go]golang.tokyo #13 - Macでハンズオンアプリを動かすまでに必要だったこと #golangtokyo"
date= 2018-03-14T09:46:52+09:00
draft = false
slug = ""
categories = ["go", "report"]
tags = ["golang", "gomobile","android","golangtokyo"]
author = "budougumi0617"
+++



golang.tokyo #13の参加メモ。今回のgolang.tokyoはgomobileハンズオンだった。
自分は今回半分運営お手伝いとして参加していて、前半にあった技術の説明は聴けていないので、ハンズオンでデモアプリを動かすまでに必要だったことをメモしておく。

|||
|---|---|
|URL|https://golangtokyo.connpass.com/event/79039/|
|会場|株式会社Gunosy(六本木ヒルズ森タワー25階)|
|日時|2018/03/09(金)19:10 〜 22:00|
|資料|[gomobileハンズオン](https://docs.google.com/presentation/d/1rHwNMweNlqohkAsUlAwWEZx5wxcRm10KR1p5UxTPaaA/edit#slide=id.p)|


# gomobileとは
https://github.com/golang/go/wiki/Mobile

gomobileはGoのコードをAndroid(Java)/iOS(Obje-C/Swift)向けのライブラリとしてビルドするツール。

# ハンズオンの内容
https://github.com/golangtokyo/gomopost

今回はGoで書かれたチャットクライアントAPIライブラリをAndroidから呼ぶハンズオン内容だった。
基本的にコードは全て用意されていて、Android Studioでデバッグ実行すればよいだけなのだが、
Androidのデバッグ環境を整えるまでがだいぶ難関だった（といってもGoは慣れているだけだったかもしれない。

# 前提
自分は以下の状態のMacで臨んだ。

- GoのもろもろGOPATHなりPATH通しなりの環境構築は済んでいる
- Android関係の環境構築は当日まで一切やってない
- 諸々の事情でMacのJavaの環境はだいぶ汚れている

また、今回のハンズオンで使う`golangtokyo/gomopost`で指定されているAndroidのSDK versionは26。  
https://github.com/golangtokyo/gomopost/blob/2d55f51ad3c9e30b1841452354303a053672254f/android/app/build.gradle#L3-L12
```
android {
    compileSdkVersion 26
    defaultConfig {
        applicationId "io.golangtokyo.gomopost"
        minSdkVersion 22
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    ...
```

# Android側の準備
まずAndroid Studioの環境構築を行っておく。

## Android Studioのインストール
Android Studioのインストールページは以下。  
https://developer.android.com/studio/install.html

Install中にSDKのバージョンを選択できる場合は26(Android 8.0(Oreo))を選択しておく（自分はここで27をインストールしてしまった）。  
インストールが終わったらAndroid Studioを起動してアップデートを走らせておく

## Android SDK/NDKのインストール
Android Studio(Intelli J)は「Cmd+Shift+A」でいろいろ検索ができる。  
起動しているAndroid Studioで「Cmd+Shift+A」を実行して検索ウインドウをだしたら「SDK Location」を検索する。  
こうすると「Preference」ウインドウ内の「Android SDK」項目が開く。ここで以下を実施する

- あとで使うので、「SDK Location」を覚えておく（Macは「/Users/${USER}/Library/Android/sdk」みたいな感じだと思う）
- 「SDK Platform」タブ内の「Android 8.0(Oreo)」がインストールされていなかった場合はインストールする
- 「SDK Tools」タブ内の「NDK」をチェックしてインストールしておく

## $ANDROID_HOMEの設定
$ANDROID_HOMEという環境変数が設定されている必要がある。これは前述の「SDK Location」と同じでよいので、そこに書いてあったPATHを指定しておく。

```bash
export ANDROID_HOME=/Users/budougumi0617/Library/Android/sdk
```

## エミュレーターの準備
次にデバッグ用のAndroidエミュレータを作っておく。

- 起動しているAndroid Studioで「Cmd+Shift+A」を実行して検索ウインドウを出す
- 「AVD Manager」を検索して「Andorid Virtual Device Manager」ウインドウを表示する
- 「Create Virtual Device...」ボタンをクリックして、API 26のエミュレータを作成する

エミュレータにはそこそこマシンスペックが必要になるので、自分は「Nexus 5X」で作った。

## Javaのライブラリの状態をキレイにしておく。
後々`gomobile bind`コマンドをするときに自分の汚れた環境だとハマった。もろもろのJARファイルを裏でごにょごにょしているらしく、
ローカルに入っているJARファイルの権限をキレイにしておく必要がある。自分のPCでは以下を行った。

```bash
$ echo $USER
budougumi0617
$ sudo chown budougumi0617 /Library/Java/Extensions/*.jar
```

## Goの準備
ここまでくればあとは資料とgomopostのREADME通りにやれば大丈夫。

- [gomobileハンズオン](https://docs.google.com/presentation/d/1rHwNMweNlqohkAsUlAwWEZx5wxcRm10KR1p5UxTPaaA/edit#slide=id.p)
- https://github.com/golangtokyo/gomopost

```bash
# gomobileをダウンロードしてくる
$ go get -u golang.org/x/mobile/cmd/gomobile

# 初期化する。$ANDROID_SDKは前述した$ANDROID_HOMEと同じなので、
# /Users/budougumi0617/Library/Android/sdk/nkd-bundleのようになる。
$ gomobile init -v -ndk=$ANDROID_SDK/ndk-bundle

# サンプルコードをgo getしてディレクトリを移動する
$ go get -u github.com/golangtokyo/gomopost
$ cd $GOPATH/src/github.com/golangtokyo/gomopost

# Android用のAARファイルが出来るか確認
$ gomobile bind -v github.com/golangtokyo/gomopost
```

これでうまくいくならば、あとは以下でOK

- `golangtokyo/gomopost`ディレクトリで`make`コマンドを実行する
- Android Studioを開いて、「File」メニューから「Open」で`golangtokyo/gomopost/android`ディレクトリを開く
- Android Studio上で右上の緑色の三角ボタンをクリック or 「Ctrl+R」を実行する
- 作成したAPI 26のエミュレータを選んでデバッグを開始する


そうすると、チャットアプリが起動するはずだ
（チャットの動作確認にはチャットサーバが必要）


## 終わりに
Androidはほとんど触ったことなかったので、だいぶ手間取った。
当日はSDKまではダウンロードしていたのだが、NDK、適切なAPIレベルのSDKを再インストールでハンズオンの時間をほとんど使ってしまった。事前準備何事も大事。

とはいえGo側の準備はそれほどめんどくさく無いはずなので、モバイルアプリ作成者がGoのライブラリを取り入れるのはそれほどハードルが高くなさそう。サーバーサイドでよく話題になるGoだが、モバイルにも活躍の場が広がっていくとおもしろいなあ




