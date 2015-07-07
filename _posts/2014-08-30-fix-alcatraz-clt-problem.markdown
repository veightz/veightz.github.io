---
layout: post
title: Alcatraz 遇到 Command Line Tools 错误解决方案
date: 2014-08-30 14:07:24
categories:
---

Alcatraz 是一款Xcode5下的图形化包管理工具, 可以方便的安装各种Xcode扩展.

# 问题描述

安装完Alcatraz后, 打开Package Manager时,弹出错误
> Xcode Command Line Tools are not currently installed, and are required to run Alcatraz.

要么你真没装Command Line Tools, 要么就是你装了但是没识别出来.
窝就是属于后者, 我还以为我Command Line Tools没装好, 反反复复装了好几次.

# 解决方案

## 安装Command Line Tools

如果你真没有安装Command Line Tools, 请看本节,否则直接看下一节

### 方法一: 命令行安装

{% highlight sh %}
xcode-select --install
{% endhighlight %}

### 方法二: 去开发者中心下载DMG包安装

> <https://developer.apple.com/downloads/>


## 修复git

我遇到这个问题,是因为昨晚升了git的版本(使用brew cask, [点击查看详情](/2014/08/30/brew-cask/), 然后git的路径就变了,导致问题的出现.Alcatraz中有一行源码:
<!--more-->

{% highlight objc %}
// ATZGit.h
static NSString *const GIT = @"/usr/local/bin/git";
{% endhighlight %}

可以看到它指向了`/usr/local/bin/git`这个位置去调用git.这个位置是自带git的路径,而我更新了git后,安装的路径在`/usr/local/bin/git`, 查看自己当前git路径的方法:

{% highlight sh %}
which git
{% endhighlight %}

我的输出结果为:

{% highlight sh %}
/usr/bin/git
{% endhighlight %}

所以Alcatraz在`/usr/local/bin/git`找不到git,就报错了.解决这个问题, 我们在`/usr/local/bin/`下做个软链接, 指向我们现在的git路径,这样Alcatraz就能正确找到git了.代码如下:

{% highlight sh %}
sudo ln -s /usr/local/bin/git /usr/bin/git
{% endhighlight %}

> More Details:<https://github.com/supermarin/Alcatraz/issues/152>
