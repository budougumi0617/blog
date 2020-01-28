+++
title= "PHPで独自クラスを使った型キャストをおこなう方法"
date= 2020-01-29T07:29:29+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["php"]
tags = ["php"]
keywords = ["php","PHP","キャスト"]
twitterImage = "logos/php.png"
+++

PHPで型を使って安全かつIDEの支援を受けながら開発したい。  
`array`やインターフェイスで受け取った引数に、型をキャストする方法を考えた。

<!--more-->

# TL;DR
- PHPはスカラー型や`object`型を使ったキャストしかサポートしていない
- 戻り値に型を指定した関数を用意すれば独自型のキャストができる
- PHP7.4からはアロー関数を使った簡易関数も作れる
- 型を指定しておくとIDEに支援もフルに受けることができる

```php
final class CastObject extends ParentObject
{
    // 型キャストを行なうだけの関数
    public static function cast($obj): self
    {
        if (!($obj instanceof self)) {
            throw new InvalidArgumentException("{$obj} is not instance of CastObject");
        }
        return $obj;
    }
}

// アロー関数を使ったキャスト関数
$cast = fn($orig): CastObject => $orig;
```

# 独自クラスへの型キャストがしたい。
`php-go`というPHP製のGoインタプリタを作っている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/php-go" data-iframely-url="//cdn.iframe.ly/SfTZMuB"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

インタプリタの実装をしていると、基となるインターフェイス（あるいは`array`）としてオブジェクトを受け取り、再帰的な実際の具象クラスの種類ごとに処理を切り替えるような実装を行なう。  
そのような処理を再帰的に繰り返すため、少しでも意図しないロジックパスを通りそうになったら（期待する具象クラスではなかったら）即例外を発生させたい。  
また、IDEの支援をフルに受けるためには、きちんと型をキャストしたほうがよい。

# 独自型をキャストする方法
`$casted = (MyClass)$obj;`のように書けば終わり、だったら簡単なのだが、PHPのキャストはスカラー型や`object`型でしか利用できない。

- 型キャスト - Manual - PHP
	- https://www.php.net/manual/ja/language.types.type-juggling.php#language.types.typecasting

## キャスト関数をつくり戻り値型を指定する
ではどうすればいいかと言うと、関数の戻り値に型を指定したキャスト用の関数を用意するのが一番簡単そうだった。

- PHP Cast to my class - Stackoverflow
	- https://stackoverflow.com/a/54481076/12802174

たとえば、こんな継承関係の`ParentObject`クラスと`CastObject`クラスがあったとする。

```php
class ParentObject{}

final class CastObject extends ParentObject
{
    public string $prop;

    public function __construct(string $prop)
    {
        $this->prop = $prop;
    }
}
```

あらかじめ、キャスト用の関数を用意しておく（多用するならば、クラスに`static`関数として用意すればよいだろう）。
```php
final class CastObject extends ParentObject
{
    public static function cast($obj): self
    {
        if (!($obj instanceof self)) {
            throw new InvalidArgumentException("{$obj} is not instance of CastObject");
        }
        return $obj;
    }
}
```
どこからか受け取った`ParentObject`オブジェクトに`CastObject`クラスをキャストしたいならば、以下のように書けばよい。
```php
// どこからか受け取ったParentbjectな $obj
if ($obj instanceof CastObject) {
    $casted = CastObject::cast($obj);
}
```

## 7.4からはアロー関数も利用できる
PHP7.4からはアロー関数という無名関数の書き方が用意されている。

- アロー関数 | PHP 7.3.x から PHP 7.4.x への移行 - Manual - PHP
	- https://www.php.net/manual/ja/migration74.new-features.php#migration74.new-features.core.arrow-functions

何度も使わないようなキャストならば、その場で使い捨ての関数として宣言してもよいだろう。

```php
$cast = fn($orig): CastObject => $orig;

// どこからか受け取ったParentbjectな $obj
if ($obj instanceof CastObject) {
    $casted = CastObject::cast($obj);
}
```

# 終わりに
明示的に型がわからなくてもメソッド呼び出しなどはエラーにならない。  
ただ、IDEのサポートを受けたいならば明示的に型情報が付与されていたほうがよい。  
また、型が自明だと安心して処理を続けられる。

# 参考
- 型キャスト - Manual - PHP
	- https://www.php.net/manual/ja/language.types.type-juggling.php#language.types.typecasting
- アロー関数 | PHP 7.3.x から PHP 7.4.x への移行 - Manual - PHP
	- https://www.php.net/manual/ja/migration74.new-features.php#migration74.new-features.core.arrow-functions
- PHP Cast to my class - Stackoverflow
	- https://stackoverflow.com/a/54481076/12802174
