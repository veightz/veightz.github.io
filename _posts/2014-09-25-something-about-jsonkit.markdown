---
layout: post
title: Direct access to objective-c's isa is deprecated in favor of object_setClass() and object_getClass(). 解决办法
date: 2014-09-25 23:50:11
categories:
---
今天队友引进了JSONKit这个第三方的JSON解析库.于是20枚编译错误GET.不过队友没遇到问题和头头pull下来也直接跑.后来发现他们跑了4s的模拟器,而我在跑5s的模拟器.所以初步推测是64位平台送来的彩蛋.
稍微看了下错误原因,基本都是一个错误.
> Direct access to objective-c's isa is deprecated in favor of object_setClass() and object_getClass().

修改起来其实很简单,就是体力活.
两个简单的例子就能说明.

{% highlight c %}
// Setter
// 报错形式
objct->isa = _JKArrayClass;
// 修改方法
object_setClass(object,_JKArrayClass);

// Getter
// 报错形式
_JKArrayClass = objct->isa;
// 修改方法
_JKArrayClass = object_getClass(object);
{% endhighlight %}

简单的说就是不能再用原来object->isa去get或者set了,而是需要自己手动调用getter或setter.
