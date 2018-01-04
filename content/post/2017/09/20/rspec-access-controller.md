+++
title= "Rspec内でテスト対象のControllerのメソッドの戻り値をスタブする"
date= 2017-09-20T08:52:42+09:00
draft = false
slug = ""
categories = ["RubyOnRails"]
tags = ["RubyOnRails","Test", "RSpec"]
author = "budougumi0617"
aliases = ["/post/2017/09/20/rspec-access-controller/"]
+++

`C#`では対象インスタンスのメソッドの挙動を変えることは出来ないので、別の手段を探していたのだが、`ruby`では出来た。

# TL;DR
- RSpec書いたControllerSpecの中でテスト対象のコントローラのメソッドの戻り値をモックオブジェクトに変えたかった。
- `ControllerExampleGroup`で定義されている`controller`からテスト中のコントローラインスタンスを操作することが可能

[Module: RSpec::Rails::ControllerExampleGroup#controller](http://www.rubydoc.info/gems/rspec-rails/RSpec/Rails/ControllerExampleGroup#controller-instance_method)


# 前提
`rspec-rails (~> 3.0.0.beta2)`で確認。

テスト対象のコントローラはこんな感じ。

```ruby
class MyController < ApplicationController
  def my_client
    MyClient.new
  end
end
```

# `controller`と`allow`でメソッドの戻り値をすげ替える
`allow`を使えばメソッドの挙動をすげ替えることが出来る。`Rails`の場合は`controller`を使えばテスト対象のコントローラのインスタンスを得られるので、テストを実行する前に下記の操作をしておくことで、テスト対象のメソッドの挙動も変更しておくことができる。下記の例は、`double`で作成したダミーオブジェクトで、`MyController#my_client`の挙動を変更する例。例外を出す例も記載している。

```ruby
# MyControllerのRSpecのdescribe内

# ダミーオブジェクト作成
my_client_mock = double('Dummy Client')
# メソッドの振る舞いを設定しておく。
allow(my_client_mock).to receive(:my_method).and_return("foo")

# テスト対象のコントローラのメソッドにダミーオブジェクトを設定
allow(controller).to receive(:my_client).and_return(my_client_mock) 
# 例外を出す場合
allow(controller).to receive(:my_client).and_raise(MyError.new('bar')) 
```


# 参考

[Module: RSpec::Rails::ControllerExampleGroup#controller](http://www.rubydoc.info/gems/rspec-rails/RSpec/Rails/ControllerExampleGroup#controller-instance_method)

[Rspec3のexpectとallowの違い](http://jangajan.com/blog/2014/12/08/expect-and-allow-in-rspec/)

[RSpecまとめ(2)～Mock(double/stub/mock)～](http://web-k.github.io/blog/2012/10/02/rspec-mock/)
