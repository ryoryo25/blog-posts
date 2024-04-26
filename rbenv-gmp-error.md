---
title: 'rbenv install 3.1.2で"library gmp not found"エラーが出る'
coverImage: 'error-log.png'
dates:
  postDate: '2023-12-05T18:29:14.902461'
  updateDate: null
---

# TL;DR

以下のいずれかの方法で解決可能
- `rbenv install --list`で表示されるバージョンをインストールする．
- `rbenv install 3.1.2 -- --without-gmp`でインストールする．

# 背景

`rbenv`でRuby 3.1.2をインストールしようとしたら`make`が失敗して次のようなエラーが出る．

```
...
ld: library 'gmp' not found
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [libruby.3.1.dylib] Error 1
make: *** Waiting for unfinished jobs....
...
```

`rbenv`のバージョンは以下の通り
```
$ rbenv -v
rbenv 1.2.0-82-gd10388a
```

# 解決方法

## 入れたいRubyのバージョンが固定の場合

`rbenv`のこのIssue[^1]によると`./configure`実行時に`gmp`を使用するように指定されているが，実際には使用されていないようである．なので，`rbenv install`のオプションとして`--without-gmp`を渡してやると`gmp`が使用されずにインストールできる．

```
$ rbenv install 3.1.2 -- --without-gmp
ruby-build: using openssl@1.1 from homebrew
==> Downloading ruby-3.1.2.tar.gz...
# 略...
==> Installing ruby-3.1.2...

ruby-build: using readline from homebrew
ruby-build: using libyaml from homebrew
ruby-build: using gmp from homebrew
-> ./configure "--prefix=$HOME/.anyenv/envs/rbenv/versions/3.1.2" --without-gmp --with-openssl-dir=/opt/homebrew/opt/openssl@1.1 --enable-shared --with-readline-dir=/opt/homebrew/opt/readline --with-libyaml-dir=/opt/homebrew/opt/libyaml --with-gmp-dir=/opt/homebrew/opt/gmp --with-ext=openssl,psych,+
-> make -j 8
-> make install
==> Installed ruby-3.1.2 to /Users/***/.anyenv/envs/rbenv/versions/3.1.2
```

[^1]: https://github.com/rbenv/ruby-build/issues/1709

## 入れたいRubyのバージョンが固定でない場合

`rbenv`が正しく動作するのが保証されているのは`rbenv install --list`で表示されるバージョンであると考えられるため，ここに表示されているバージョンで事足りる場合はここに表示されているバージョンを入れよう．