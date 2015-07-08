---
layout: post
title: description & debugDescription
date: 2015-07-08
---

有时候我们会在代码中直接输出对象，如：

{% highlight objc%}
NSLog(@"%@", objc);
{% endhighlight %}

原生类的效果还不错，但是自定义的类给出的信息可能只有类型名和内存地址了。

看了下头文件，在`NSObject.h`可以发现：
{% highlight objc %}
@protocol NSObject

// ...

@property (readonly, copy) NSString *description;
@optional
@property (readonly, copy) NSString *debugDescription;

@end
{% endhighlight %}

协议`NSObject`(不是类， 是协议) ，声明了`description`，`debugDescription`这两个属性，
ObjectiveC 的两个 Root Class，`NSObject`和`NSProxy`都实现了description，当代码逻辑中直接打印对象的时候，就是调用`description`。
而在`lldb`调试的时候，会默认优先调用`debugDescription`，当`debugDescription`没有实现的时候，会去调用`description`。

由于`debugDescription`是`optional`, `NSObject`并没有实现它，倒是`NSProxy`进行了实现，不过日常开发的时候，用的都是`NSObject`的子类。

测试用例：
{% highlight objc %}
- (NSString *)description {
    return NSStringFromSelector(_cmd);
}

- (NSString *)debugDescription {
    return NSStringFromSelector(_cmd);
}
{% endhighlight %}
