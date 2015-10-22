---
layout: post
title: "关于惰性初始化的随想"
date: 2015-10-21 22:33:44
categories:
---

惰性初始化是开发中很常见的一种模式，但是许多细节都值得思考。

# 怎么初始化

常见的惰性初始化方法分为两种。一种是使用dispatch_once_t做初始化的标志位，一种是判断实例变量是否为 nil。场景各有不同。

使用dispatch_once_t做初始化的标志位的方法主要用于一些单例的生成，而且这种类往往不是视图，在应用的生命周期中多次使用，不是意外情况下不会被释放。
同时这样的方法是线程安全的。

{% highlight objc %}
+ (DataManager *)sharedManager {
    static DataManager *sharedManager = nil;
    static dispatch_once_t onceToken
    dispatch_once(&onceToken, ^{
        sharedManager = [[self alloc] init];
    });
    return sharedManager;
}
{% endhighlight %}

另一种判断实例变量是否为 nil 的方法常见于对象的属性的初始化。

{% highlight objc %}
- (UIView *)aView {
    if (!_aView) {
        _aView = [[UIView alloc] init];
        /* 一些配置逻辑 */
    }
    return aView;
}
{% endhighlight %}

对于视图对象来说，使用实例变量是否为空来判断会更加方便。不用过多的考虑在什么位置放置初始化的逻辑，也不用有意去处理属性为空的逻辑。
虽然逻辑上有非线程安全的问题，但是在 iOS 的平台上，UI 操作会在主队列进行，主队列又是串行单线程，所以不用担心这个问题。

# 初始化方法的职责范围

经常看到一些代码，在 getter 方法中的初始化过程中做了很多很多事情。比如在刚刚的例子里，

{% highlight objc %}
- (UIView *)aView {
    if (!_aView) {
        _aView = [[UIView alloc] init];
        /* 一些配置逻辑 */
        // ...

        /* 多了视图的添加 */
        [self.view addSubview:_aView];
    }
    return aView;
}
{% endhighlight %}

乍一看也没什么啊，好像还挺方便的样子。（要是 frame 逻辑不多，直接设定好的话。）

如果 Memory Warning 来了呢？

{% highlight objc %}
- (void)didReceiveMemoryWarning{
    [super didReceiveMemoryWarning];
}
{% endhighlight %}

如果不进行 _aView = nil 的处理，要是 self.view 被释放了，那么 _aView.superview 也就为 nil 了。
可是 _aView 还在（也说明没被释放掉），那么所有的 self.aView 直接返回 _aView 。但是 _aView 也不在之后的 self.view 上。
可能一时之间变成诡异的 bug 。

{% highlight objc %}
- (void)didReceiveMemoryWarning{
    [super didReceiveMemoryWarning];
    _aView = nil;
}
{% endhighlight %}

那么好，我加上指向 nil 的处理行了吧。要是 self.view 最后没被释放，那么 _aView 作为 self.view.subviews 被继续持有着。
当下一段 self.aView 出现时，一个克隆体就盖了上去。

在一些特别的情况下，可能会出现下面的代码：

{% highlight objc %}
- (UIView *)xView {
    if (!_xView) {
        _xView = [[UIView alloc] init];
        /* 一些配置逻辑 */
        // ...

        /* 多了视图的添加 */
        [self.yView addSubview:_xView];
    }
    return aView;
}

- (void)didReceiveMemoryWarning{
    [super didReceiveMemoryWarning];
    [_xView removeFromSuperview];
}
{% endhighlight %}

在某种机缘巧合下，_xView 顺利从 _yView 中被移除，但是自己没有被释放。于是再也没有重新被添加回去的机会了。。。

这个例子可能不太好，只是想说明，在只有创建时才会走的逻辑里，不要添加过多非创建时也可能做的逻辑。

# 如何判断属性为空

理论上来说，这样的 getter 方法，你永远不会拿到 nil，如果拿到了，大概是你没写好吧。。。。。。
可是有时候就是要判断下某个视图是不是为空，或者一不小心访问个某视图的某属性，然后把它给创建了。
（如果此时还遇上初始化逻辑里做了很多别的事情的话。。。）

总的来说，有时候需要安全地判断下是否为空，就很麻烦。
如果是自己的属性，那还好办，直接用实例变量访问，不走 getter 方法的逻辑就行。
如果某个属性是别人家的孩子，或者是自己父类家的孩子，那就不好办了。

举个例子，还是 Memory Warning 中的处理， 一般来说，都是要判断下当前视图控制器的 view 是不是正在显示的。

{% highlight objc %}
- (void)didReceiveMemoryWarning{
    if (!(self.isViewLoaded && self.view.window)){
    }
    [super didReceiveMemoryWarning];
}
{% endhighlight %}

self.view.window 为 nil 时，不是正在显示。为什么不直接 self.view.window 判断呢？
没错，这个 view 也是用惰性加载的方式实现的。可以自测或者看文档。

>If you access this property and its value is currently nil, the view controller automatically calls the loadView method and returns the resulting view.

如何不意外初始化 self.view 呢？
文档中也给了方法

>Because accessing this property can cause the view to be loaded automatically, you can use the isViewLoaded method to determine if the view is currently in memory. Unlike this property, the isViewLoaded property does not force the loading of the view if it is not currently in memory.`

其实做了类似 viewController->_view 的事情。也说明了，判断这个 view 是不是为 nil 。在实践中有这个需要在， 所以给了这个方法来判断。

所以说，如果是暴露给外部的属性，使用了惰性加载，就要好好考虑下，是不是该给个 isXXLoaded 方法了，或者在头文件中做明确的说明。
