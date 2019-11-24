+++
title= "[書評] 気づけばプロ並みPHP改訂版を読んでPHPに入門した"
date= 2019-11-25T06:10:08+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["review", "php"]
tags = ["php","book"]
keywords = ["気づけばプロ並みPHP","書評","PHP"]
twitterImage = "2019/11/25_pronami_php.png"
+++

「気づけばプロ並みPHP 改訂版--ゼロから作れる人になる!」を読んでPHPの勉強を始めた。  
ECサイトの基本的な機能を実装しながら一通り読み終わったので感想を書く。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4865940650&linkId=651b493478cd53b1c985d9c1620a9e07"></iframe>
<!--more-->

# 所感
純粋なPHPとRDBMSを使ってWEBサイトを構築するのが本書だ。  
リクエストパラメータへのアクセス、セッション処理、リダイレクトなどひととおりのWEBサービスの処理と、データベースを使った永続化の実装ができる。  
それぞれに関する機能や構文の解説も載っており、ほとんどプログラミング経験がない人でも、楽しくWEBサービスを構築しながら、PHPとWEBサービス作成時に必要な周辺技術の勉強ができるのではないだろうか。  
他言語をそれなりにやっている私としてはだいぶ初歩的な内容だった。サンプルコードが初心者向けの単純ロジックだったので、リファクタリングをして関数化をしたり、言語仕様を読みながら楽しくすすめることができた。


# どんな本なのか
この書籍はプログラミング初心者向けのPHP学習本で、ECサイトの作成を通して基本的なPHPの使い方を学ぶ。  
「PHPの言語仕様・文法を体系的に学ぶ」というよりは「実際に動くサービスを作ってみて、その時どんなPHPの機能を使えるのか」という本だ。  
書籍をひととおり写経すると、主に以下の学習・機能の作成を行なうことができる。

- XAMPP環境の構築
- データベースを使ったスタッフ登録・一覧処理
- 画像を表示する商品一覧の作成
- セッションを使ったログイン機能・要認証画面の作成
- セッションを使ったショッピングカート機能の作成
- 複数テーブルを使った商品購入
- 商品購入時のメール送信処理
- データのCSVダウンロードの機能

# なぜ読もうと思ったのか
一言でいうと、ある程度の機能を書く中でPHPという言語の特徴を知りたかったので手にとった。  
私は他の言語（C/C++/C#/Java/Ruby/Go/React）の経験はあるが、今までPHPを触ったことはなかった。  
本書を読む前に、すでに[独習PHP](http://amazon.jp/dp/B01FH3KVNU)などを読んでPHPの基本的な言語仕様はなんとなく理解していた。  
それなりに動くものを作ろうとCakePHPのチュートリアルなどをやってみたが、言語仕様というよりアプリケーションフレームワークの力を借りる勉強という感じだった。  
生のPHPでどれくらいできるのか知りたかったので、フレームワークを使わずにECサイトの機能を構築する本書を読むことにした。  
写経を通してPhpStormを使うときの運指やスニペットなどを学習する目的もあった。

# 本の写経を通してわかったこと
なんとなくだが、PHPのコードの雰囲気がわかった。  
本書のコードはオブジェクト指向ではないので、実際にCakePHPやLaravelを使ってアプリを作成するときの書き味はだいぶ変わると思う。  
が、PHPのHTMLに対するテンプレートエンジン的な特徴をだいぶ実感することができた。  
とくにプロセスのリスタートなどを意識せずとも、ブラウザリロードのみで動作確認を繰り返せるのも軽快でよかった。  
`$_POST`などでリクエストパラメータにEasyなアクセスができるのもWEB向きの言語なんだなと思った。

本書は全9章になるが、8章まで写経をしたリポジトリは以下になる（9章はそれまでのコピペで終わるので省略して良い。と本文に書いてある）。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/budougumi0617/pronami_php" data-iframely-url="//cdn.iframe.ly/fmhuUUr"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

また、本書は初心者向きであること、古いPHPを想定して書かれているため、冗長な処理やPHP7では書かないような書きかたも多い。  
そういう部分は積極的に脱線して別の書き方をしたり、別の解決手段を取ったりした（ヒアドキュメントを使ったりetc）。

## 型宣言を利用する
たとえば、私はPHP7.3ベースで勉強したので、なるべく処理を関数に抽出し、引数・戻り値に型宣言を追加した。
極力`declare(strict_types=1);`も付与している。

```php
function validate(string $name, string $pass, string $pass2): bool
{
    // do validating...
}
```

## Docker Composeとマイグレーションツールを使って勉強する
本書では環境構築方法としてXAMPP環境の構築手順が記載されている。  
XAMPPを使うモチベーションはなかったのでDocker Composeなどを使って環境構築を行なった。

### Docker Composeを使った環境構築
XAMPPを入れると不要な常駐プロセスが増えそうだし、私はローカルで使うDBは基本コンテナで用意するので、すべてDockerで開発環境を用意することにした。  
PHPのDocker開発環境の構築方法は以下のブログを参考にした。  
今後CakePHPの勉強もするので、CakePHP用のコンテナがベースになっている。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fbufferings.hatenablog.com%2Fentry%2F2018%2F08%2F28%2F015335" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>
<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Ftenkoma.hatenablog.com%2Fentry%2F2019%2F03%2F24%2F023944" style="border: 0; width: 100%; height: 190px;" allowfullscreen scrolling="no"></iframe>

実際に利用したDocker環境は以下の通り。

- https://github.com/budougumi0617/pronami_php/blob/master/docker/
- https://github.com/budougumi0617/pronami_php/blob/master/docker-compose.yml

### Phinxを使ったDBマイグレーション
書籍の中では`phpadmin`を使ってDBの操作を行なっている。SQLを素で叩いてマイグレーションするのもどうかと思ったのでマイグレーションツールを使うことにした。  
PHPのデファクトはわかっていないが、CakePHPでも使われているようだったので、`Phinx`というマイグレーションツールを利用することにした。

- Phinx
    - http://docs.phinx.org/en/latest/index.html

`Phinx`は依存管理の練習がてら`Composer`を使ってインストールした。

```bash
$ docker-compose run --rm php-cli composer --dev require robmorgan/phinx
```

Phinx自体は決まりに則ってPHPのクラスメソッドにテーブル構造を記載すればよい。

```php
<?php
use Phinx\Migration\AbstractMigration;
class CreateStaffTable extends AbstractMigration
{
    public function change()
    {
        // create the table
        // How to change id from integer to biginteger: https://github.com/cakephp/phinx/issues/519
        $table = $this->table('mst_staff', ['id' => false, 'primary_key' => 'id']);
        $table->addColumn('id', 'biginteger', ['identity' => true, 'signed' => false])
            ->addColumn('name', 'string', ['limit' => 15])
            ->addColumn('password', 'string', ['limit' => 32])
            ->create();
    }
}
```

あとはマイグレーションコマンドを実行すればDBに新しいテーブルができる。
```bash
$ vendor/bin/phinx migrate
```

## 脆弱性まわりの実装について
本書の内容だと、`htmlspecialchars`関数を使ったXSS対策やプレースホルダを使ったSQLインジェクション対策程度はされているのだが、メール送信機能などまでは脆弱性対策がされていない。  
そのような危険なコードに関して、WEBセキュリティ（脆弱性）で有名な[体系的に学ぶ 安全なWebアプリケーションの作り方](http://amazon.jp/dp/B07DVY4H3M)（通称徳丸本）の著者である徳丸氏の助言が記載されている。  
私は徳丸本を未読なのだが、徳丸本もサンプルコードはPHPらしいので、早いうちに読んでおこうと思った。

# 本を読んで気になったところ
初心者用の書籍なので冗長な処理などはあっても良いと思うが、完全に動かないところがあったり、気になったところがあったので書いておく。

## MySQL5.7を使っているとテーブルのロックが不十分で動かない
「P261 7-5 もっと安全にしよう！」ではDB操作時にロックを取得するように変更を行なう。  

```sql
$sql = 'LOCK TABLES dat_sales WRITE, dat_sales_product WRITE';
```

しかし、MySQL5.7のコンテナを使ってDBを用意していたせいか、書籍に書いてあるとおりにRDBに対してロックを取得するとエラーが出た。

```php
/app/shop/shop_form_done.php:89:
object(PDOException)[2]
  protected 'message' => string 'SQLSTATE[HY000]: General error: 1100 Table 'mst_product' was not locked with LOCK TABLES' (length=88)
  private 'string' (Exception) => string '' (length=0)
  protected 'code' => string 'HY000' (length=5)
  protected 'file' => string '/app/shop/shop_form_done.php' (length=28)
  ...
```

これはロックするテーブルが不足しているためだった。MySQL5.7ではロック中READ操作のみのテーブルに対してもロック操作が必要だった。

https://dev.mysql.com/doc/refman/5.6/ja/lock-tables.html

> ロックが必要なセッションは、必要なすべてのロックを 1 つの LOCK TABLES ステートメントで取得する必要があります。このように取得されたロックが保持されている間、このセッションは、ロックされたテーブルにのみアクセスできます。

エラー内容の通り、ロックのSQL文にテーブルを追加することで解決した。

```sql
$sql = 'LOCK TABLES dat_sales WRITE, dat_sales_product WRITE, mst_product READ';
```

## 例外をすべて握りつぶしている
書籍中のサンプルコードの例外処理はすべて以下のような`try-catch`構文になっている。  

```php
try {
    // do anything...
} catch (Exception $e) {
    echo 'ただいま障害により大変ご迷惑をおかけしております。';
    exit();
}
```

例外でコードが動かなかったら、タイポしていないかよく確認しろというスタンスだ。  
が、これでは上記のように書籍の通り書いていてエラーが発生したとき、途方にくれてしまうだろう。
「このままではインターネット上に公開できません」とのただし書きも明記されているので、見た目にこだわらず`var_dump($e);`を仕込んでおいたほうが、何が問題だったのか原因究明が早くなると思う。

# 今後どう活かすのか
本書は初心者向きということでオブジェクト指向的な実装は一切行わない（少し関数化をするくらい）。  
クラスを使った実装は引き続き勉強が必要になる。また、今回は書籍で触れられていないことを理由にテストを一切書かなかった。  
業務で扱う場合は当然テストを書く必要があるので、PHPUnitなどの学習をしていこうと思う。

# 参考
- https://github.com/budougumi0617/pronami_php
- [気づけばプロ並みPHP 改訂版--ゼロから作れる人になる!](http://amazon.jp/dp/4865940650)
- [独習PHP 第3版](http://amazon.jp/dp/B01FH3KVNU)
- [TECHNICAL MASTER はじめてのPHPプロフェッショナル開発 PHP7対応](http://amazon.jp/dp/B07SPY5SXH)
- [PhpStorm+DockerでCakePHPの開発環境を作ってみた](https://bufferings.hatenablog.com/entry/2018/08/28/015335)
- [PhpStormと連携する必要最小限のDocker環境を作る](https://tenkoma.hatenablog.com/entry/2019/03/24/023944)
- [Phinx Documentation](http://docs.phinx.org/en/latest/index.html)


<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4865940650&linkId=651b493478cd53b1c985d9c1620a9e07"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B01FH3KVNU&linkId=34ea8b5c5aab3ba550807ea5f22918e9"></iframe>
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=github.io-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B07SPY5SXH&linkId=575f1fbdf1fec1c0ede623060b78ea76"></iframe>
