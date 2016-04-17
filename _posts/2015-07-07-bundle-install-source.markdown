---
layout: post
title: bundle install 安装依赖失败
date: 2015-07-07
---

今天安装 RoR ， 执行

```sh
bundle install
```

之后， 出现了一下内容。

```sh
Fetching gem metadata from https://rubygems.org/............
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...

/* ... */

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/debug_inspector-0.0.2.gem)
An error occurred while installing debug_inspector (0.0.2), and Bundler cannot
continue.
Make sure that `gem install debug_inspector -v '0.0.2'` succeeds before
bundling.
```

看了下 `Gemfile`， 这些依赖也是间接依赖。
执行了下对应的安装，挺顺畅的，没什么问题。
看了下我 `gem` 的源,

```sh
>  gem source -l
*** CURRENT SOURCES ***

http://ruby.taobao.org/
```

似乎也没问题。

我还特意去改了下 npm 的源， 指向了淘宝的仓库。似乎还是没什么 用。

然后突然发现 Gemfile 头顶一句

```sh
source 'https://rubygems.org/'
```

所以，改成这样吧：

```sh
source 'https://ruby.taobao.org/'
```

其实还没想好要不要入 RoR 的坑。
