+++
title= "Googleスライドで「いい感じ」に背景画像を設定する"
date= 2018-05-18T09:13:20+09:00
draft = false
slug = ""
categories = ["presentation"]
tags = ["slide"]
author = "budougumi0617"
twitterImage = "logos/googleslide.png"
+++

Googleスライドで「背景を画像にしていい感じ」の資料を作る方法を調べた。

# TL;DR
- 「背景」として画像を設定してはいけない
- メニューから「挿入」-「画像」を選び画像を添付する
- 画像の「書式設定オプション」を表示して明るさを暗くしていく

## Before
![before_image](/2018/05/0518_before_slide.png)

## After
![after_image](/2018/05/0518_after_slide.png)

# 「いい感じ」のスライド資料を作る

最初に書いておくと、Azusaという「いい感じ」のスライドを作るテンプレートがすでにある。

**Azusa**  
http://sanographix.github.io/azusa-keynote/

ただ、後述するブログ記事にも書いてあるがさすがに最近はAzusaだとカブる率が高そうなので、
他の人がやっているような写真背景のスライドを作りたくなった。
keynoteでの作り方は[@kakakakakku](https://twitter.com/kakakakakku)さんの記事を参考にした。

**個人的な Keynote ベストプラクティス 2017**  
https://kakakakakku.hatenablog.com/entry/2017/05/30/092247

ただ、会社ではGoogleスライドがデフォルトなので、Googleスライドで同じことをやる方法を調べた。

# 画像を探す
背景画像については上記のブログにもあったUnsplashなどで探してくる

**Unsplash**  
https://unsplash.com/

[@TAKAKING22](https://twitter.com/TAKAKING22)さんのこの記事なども参考になる。

**プレゼンテーションに使う画像の探し方**  
https://qiita.com/TAKAKING22/items/20c006206d2ce23a5608

今回はいい感じ（だけど文字が埋まりそうな）こちらの画像を使う。

https://unsplash.com/photos/nEEa3_4AS10

![building](/2018/05/joel-filipe-189099-unsplash.jpg)

<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px;" href="https://unsplash.com/@joelfilip?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Joel Filipe"><span style="display:inline-block;padding:2px 3px;"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-1px;fill:white;" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M20.8 18.1c0 2.7-2.2 4.8-4.8 4.8s-4.8-2.1-4.8-4.8c0-2.7 2.2-4.8 4.8-4.8 2.7.1 4.8 2.2 4.8 4.8zm11.2-7.4v14.9c0 2.3-1.9 4.3-4.3 4.3h-23.4c-2.4 0-4.3-1.9-4.3-4.3v-15c0-2.3 1.9-4.3 4.3-4.3h3.7l.8-2.3c.4-1.1 1.7-2 2.9-2h8.6c1.2 0 2.5.9 2.9 2l.8 2.4h3.7c2.4 0 4.3 1.9 4.3 4.3zm-8.6 7.5c0-4.1-3.3-7.5-7.5-7.5-4.1 0-7.5 3.4-7.5 7.5s3.3 7.5 7.5 7.5c4.2-.1 7.5-3.4 7.5-7.5z"></path></svg></span><span style="display:inline-block;padding:2px 3px;">Joel Filipe</span></a>

# Googleスライドで背景に画像を利用する

普通にスライドの背景に設定すると、こんな感じになる。
案の定少し文字が背景に埋もれて読みにくくなっている部分がある。

![before_image](/2018/05/0518_before_slide.png)

Keynoteの場合はこの上に黒くて半透明な「図形オブジェクト」で載せることで画像の明るさを調整できる。


![on_keynote](/2018/05/0518_on_keynote.png)

が、Googleスライドでは2018/05/18現在半透明な「図形オブジェクト」は作成できない。


また、「背景」として画像を設定すると、画像自体の調整もできなくなる。

# 「画像オブジェクト」にして「明るさ」を調整する
そこで、Googleスライドの場合は「画像オブジェクト」として背景にしたい画像をスライドに添付する。
画像自体の大きさはマウスドラッグで調整していると、スライド端とマッチしたときにガイドが赤くなるのでピッタリのサイズで添付することができる。


![upload_image](/2018/05/0518_upload_image.png)

「画像オブジェクト」を右クリックして「順序」から「最背面」を選んで実質「背景」にしておく。

![set_layer](/2018/05/0518_set_layer.png)

あとは「画像オブエジェクト」を選択した状態で「表示形式」（or右クリック）から「書式設定オプション」を開く。
「調整」の中に「明るさ」という項目があるので、明るさをだいたい「-50%」から「-60%」にする。
（「明るさ」ガイドが選択状態ならば十字キーで微調整もできる）

![change_blight](/2018/05/0518_change_blight.png)

これで字が埋もれない程度の主張に背景画像を調整することができる。


![after_image](/2018/05/0518_after_slide.png)

# 終わりに
このブログもそうだが、アウトプットしてフィードバックを得るのが大事。
内容勝負なのはもちろんだが、最小限の手間でひと目を惹く発表スライドを作っていけるようになりたい。

…と、ここまで書いたが次に控えている発表はGoの発表なので、Goのスライドマスターを使うぞ！！

**Go's New Brand**  
https://blog.golang.org/go-brand

**Go Slide Masters (Google Slides)**  
https://golang.org/s/presentation-theme

# 関連
- [GoogleスライドやGoogleドキュメントのバージョン管理をする](/2018/11/28/version-control-google-file)

