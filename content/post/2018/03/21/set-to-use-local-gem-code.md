+++
title= "[Ruby]ローカルで修正したgemのコードをbundle installする"
date= 2018-03-21T14:21:32+09:00
draft = false
slug = ""
categories = ["RubyOnRails", "Ruby"]
tags = ["RubyOnRails","Bundler", "gem"]
author = "budougumi0617"
+++
Railsのプロダクトが参照しているgemを修正しながら開発する必要が出てきた。  
ローカルで編集して未コミット状態のコードを含んだの`gem`を`bundle install`する方法をまとめる。

# TL;DR
- `bundle install`するときローカルで修正中の`gem`のコードを取り込むようにしたい。
- その1 Gemfileの設定に`path: '../path/to/target_gem'`を追加する
- その2 `bundle config`を使う

# やりかた その1 Gemfileを変更する
変更した`Gemfile`をコミットしないように注意できるならば`Gemfile`を直接変更する。  
ローカルのコードを参照するには`Gemfile`内の対象の`gem`のインストール定義に`path:`を追加してファイルパスを与えるだけ。  
`git:`でリポジトリを明示的に参照している場合はその参照は消しておく。

```diff
# Gemfile内にあるローカルのコードを参照してほしいgemの定義
-  gem 'target_gem', git: 'ssh://git@github.com/USER/target_gem.git'
+  gem 'target_gem', path: '/path/to/target_gem'
```

あとは通常通りにgemを再取得する。
```bash
$ bundle clean
$ bundle install --path vendor/bundle
```

# やりかた その2 bundlerの設定を変更しておく
bundlerの設定で、ローカルのコードを見るようにしておくことができる。  
`Gemfile`があるディレクトリで以下のように行う。
```bash
$ bundle config --local local.target_gem  /path/to/target_gem
```

`gem`のコードの編集状態のチェックが有効になっているとめんどくさいのでoffにする。
```bash
$ bundle config --local disable_local_branch_check true
```

あとは「その1」同様に通常通りにgemを再取得する。
```bash
$ bundle clean
$ bundle install --path vendor/bundle
```

こちらの場合、用がすんだらbunderの設定を元にもどすこと
```
$ bundle config --delete local.target_gem
$ bundle config --delete disable_local_branch_check
```

# 終わりに
`Gemfile`の変更をコミットしないように気をつけるのか、終わった後`bundle config`を忘れないように気をつけるのか、どちらのほうがラクかは個人のお好みで。  
今設定してある`bundle`の設定は`bundle config`で確認できる。また、`bundle config --delete`はglobalな設定かlocalな設定かは見てくれないみたい。。。
```
$ bandle config --help
...
Executing bundle config --delete <name> will delete the configuration in both local and global sources. Not compatible with --global or --local flag.
...
```


# 参考
**Work against local gems without modifying your Gemfile**  
https://coderwall.com/p/tqdrhq/work-against-local-gems-without-modifying-your-gemfile

**bundle config**  
http://bundler.io/v1.16/bundle_config.html

**bundle config ローカルGitリポジトリ**  
http://ruby.studio-kingdom.com/bundler/bundle_config/#local_git_repos




