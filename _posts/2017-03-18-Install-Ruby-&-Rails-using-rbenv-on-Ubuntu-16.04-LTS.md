---
layout:   post
title:    "Install Ruby & Rails Using rbenv On Ubuntu 16.04 LTS"
date:     2017-03-18 00:21:00 +0900
tags:     ubuntu ruby rails
author:   junwoo
---

### 의존성 패키지 설치

```console
$ sudo apt-get update
$ sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev
```


### rbenv 설치 & 설정

```console
$ cd
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
$ exec $SHELL

$ git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
$ echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
$ exec $SHELL
```


### Ruby 설치

* ruby version은 사용하고자 하는 version으로 설치를 한다.
여기서는 version 2.3.3을 설치한다.
설치 후 version 2.3.3을 default로 설정한다.

```console
$ rbenv install -v 2.3.3
$ rbenv global 2.3.3
```

* ruby version 확인

```console
$ ruby -v
```

* gem을 설치할때 마다 관련 문서를 함께 받는 것이 default이다.
그러나 우리는 이 문서를 보지는 경우가 드물다. 받지 않도록 한 후, bundler를 설치하자.

```console
$ echo "gem: --no-document" > ~/.gemrc
$ gem install bundler
```

### Rails 설치

* 최신 버전의 rails를 설치한다.

```console
$ gem install rails
$ rbenv rehash
```

* rails version 확인

```console
$ rails -v
```
