---
layout: post
title: "不要使用宏定义去声明一个常量"
date: 2015-07-27 22:19:43
categories:
---

前几天追踪了一个线上的crash, 点击cell 上的按钮就会崩溃.由于项目庞大, 花了两天才重现问题.
不过问题比较容易解决. 开发人员给 UIButton 打了 tag, 在处理相应的 Action 回调时, 进行了判别.

可惜 tag 赋值太大了, 而 tag 是 NSInteger 类型,

{% highlight objc %}
#if __LP64__ || (TARGET_OS_EMBEDDED && !TARGET_OS_IPHONE) || TARGET_OS_WIN32 || NS_BUILD_32_LIKE_64
typedef long NSInteger;
typedef unsigned long NSUInteger;
#else
typedef int NSInteger;
typedef unsigned int NSUInteger;
#endif
{% endhighlight %}

在 C 语言中, int 始终是4个字节的, 而 long 在32位平台下是4字节, 64位下是8字节的, 虽然不清楚为什么不直接把 NSInteger 定义为 long 的别名.

问题修复比较容易,

{% highlight objc %}
//#define kSelfPickUpButtonTag 201506240754
static NSInteger kSelfPickUpButtonTag = 1506240754;
//#define kAgentReceiveButtonTag 201506240755
static NSInteger kAgentReceiveButtonTag = 1506240755;
{% endhighlight %}

由于 2^31 = 2147483648, 201506240754 和 201506240755 会在32位在溢出.
又由于开发的同学用了宏定义, 编译器也没有吐槽, 就容易忽视掉, 如果代码写成下面这个样子,

{% highlight objc %}
static NSInteger kSelfPickUpButtonTag = 201506240754;
static NSInteger kAgentReceiveButtonTag = 201506240755;
{% endhighlight %}

编译器还是会报 warning的, 容易提早发现问题.

说起来 Swift 中, 赋值溢出就直接炸了呀...
