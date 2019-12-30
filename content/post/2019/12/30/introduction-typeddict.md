+++
title= "[Python3.8] TypedDictとPyCharmを使うと型ヒントの圧倒的な恩恵を享受できる"
date= 2019-12-30T11:01:18+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["python"]
tags = ["python", "type hints"]
keywords = ["Python3", "Python3.8", "TypedDict", "Type Hints"]
twitterImage = "logos/python.png"
+++

業務でも趣味でもPythonを書くときはPython3.8を使っている。  
Python3.8から使える`TypedDict`とPyCharmを組合わせた開発体験が素晴らしいので紹介する。

<!--more-->

# TL;DR
- `TypedDeict`はPEP589で提案され、Python3.8から使える型ヒント
    - https://www.python.org/dev/peps/pep-0589/
- 辞書型のキーとセットをクラスとして厳密に定義できる
- PyCharmと組み合わせると以下の恩恵が受けられる
    - オブジェクト生成時に不足キーがわかる
    - Keyを設定するときに型が異なることがわかる
    - Valueアクセスの際にキーが存在するかわかる
    - Valueの型情報に対して補完が可能

```python
class UserDict(TypedDict):
    user_name: str
    email: str
```

なお、この記事は以下の環境を使っている。

- Python 3.8.0
- Pycharm 2019.3.1 PROFESSIONAL

# 事前準備
Python3.8の実行環境は次のように用意する。  
まず、`pipenv`や`pyenv`を事前に準備し、テスト用のディレクトリを作って`pipenv`でPython3.8環境を作っておく。

```bash
# 私はMacなのでbrewでインストール
$ brew install pipenv pyenv
$ pipenv --python 3.8
```

Pipfileが生成できたら、PyCharmで対象ディレクトリを開く。  
PycharmのPython実行環境は、Preferenceの中から「Project: xxx」→ 「Project Interpreter」を開いてPython3.8を認識している状態にしておく。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://pleiades.io/help/pycharm/pipenv.html" data-iframely-url="//cdn.iframe.ly/OPsqqHm?iframe=card-small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# TypedDictを使ったクラス定義とPyacharm
最近のPythonは型ヒントを使って型付けをしているかのようにコーディングができる。

- typing --- 型ヒントのサポート
  - https://docs.python.org/ja/3/library/typing.html

`TypedDict`はPEP589でPython3.8から追加された型ヒントだ。`TypedDict`を使うことで辞書型の型ヒント内容をかなり厳密に定義できる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.python.org/dev/peps/pep-0589/" data-iframely-url="//cdn.iframe.ly/cY6zGp1"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

今回はサンプルとして以下の`UserDict`クラスを定義した。

```python
class UserDict(TypedDict):
    """Typed User definition."""

    user_name: str
    email: str
```
このクラスを使うと、どれくらいの型ヒント情報が得られるのかPyCharmを使って検証する。

## キーが不足していることを検知できる
まず、インスタンス生成時にキーの指定が不足していることを検知できる。  
以下のような初期化は`email`キーの初期化が不足していると警告が出る。

```python
user = UserDict(
    user_name='John Due,
)  # Parameter 'email' unfilled
```

![unfilled](/2019/12/30_unfilled.png)

なお、このキーの不足の許可はクラス宣言時の`Tottaly`オプション（`total=False`）でオフにすることもできる。

- Totality
    - https://www.python.org/dev/peps/pep-0589/#totality

```python
class Movie(TypedDict, total=False):
    name: str
    year: int
```

## キーの型が間違っていることを検知できる
キーをすべて満たした初期化をしても、間違った型で要素を初期化していると警告される。

```python
user2 = UserDict(
    user_name=20,  # Expected type 'str', got 'int' instead
    email='user@exmaple.com',
)
```

![type error](/2019/12/30_type_error.png)

## 定義していないキーへの参照を検知できる
`TypedDict`を使ったクラスのインスタンスに対して、宣言時に定義しなかったキー名を指定すると警告を受ける。  
辞書型にすると`ドット`でフィールドを呼び出せなくなるが、これでタイポに気づかずエラーを出してしまうことはないだろう。

```python
print(user['phone'])  # TypedDict "UserDict" has no key 'phone'
```

![key error](/2019/12/30_key_error.png)

## Valueの型情報を使って補完ができる
すこし拡張した以下の`UserDict`を用意する。

```python
from datetime import datetime

class UserDict(TypedDict):
    """Typed User definition."""

    user_name: str
    email: str
    created: datetime
    updated: datetime
```

`created`キーは`datetime`型なので、`UserDict`インスタンスを使って`user['created'].`と書くとそのまま`datetime`型のメソッド一覧が補完される。


![method completion](/2019/12/30_completion.png)


## 終わりに
`TypedDict`は型ヒントなので、違反していても実行は出来てしまうし、IDEを使っていないと恩恵を受けられない。  
しかし、Pythonというと「ゆるふわなスクリプト言語」のイメージしかなかったので、ここまで厳密にできるのは驚いた。


# 参考
- [Pipenv環境 - 公式ヘルプ | PyCharm](https://pleiades.io/help/pycharm/pipenv.html)
- [typing --- 型ヒントのサポート](https://docs.python.org/ja/3/library/typing.html)
- [PEP 589 -- TypedDict: Type Hints for Dictionaries with a Fixed Set of Keys](https://www.python.org/dev/peps/pep-0589/)
- [mypyとTypedDictとtotalオプションについて - podhmo's diary](https://pod.hatenablog.com/entry/2018/08/23/222814)
