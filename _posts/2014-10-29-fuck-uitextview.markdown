---
layout: post
title: 一个UITextView不回调tableView:didSelectRowAtIndexPath:的坑
date: 2014-10-29 13:22:12
categories:
---

我做了一个UITableView, 里边是定制的Cell, Cell中间是一个UITextView.根据需要就能想到, 这个UITextView是不允许用户编辑, 选择, 没有任何视觉变化. 本意是想点击了TestView区域后回调tableView:didSelectRowAtIndexPath:事件的,于是楼主就这么写了:

{% highlight objc %}
UITextView *textView = [[UITextView alloc] init];
[self.contentView addSubview:textView];
[textView setSelectable:NO];
{% endhighlight %}
然后我就悲剧了.根本不会回调tableView:didSelectRowAtIndexPath:啊! 搞的窝中饭都没吃饱.

# 解决方法

## 方法一

由于中饭没吃好, 没有暖宝, 所以没思淫欲, 开了下脑洞.
正确的射定姿势:

{% highlight objc %}
[textView setSelectable:YES];
[textView setEditable:NO];
[textView setUserInteractionEnabled:NO];
{% endhighlight %}

## 方法二

在stackoverflow上看到的
[点击查看](http://stackoverflow.com/questions/1426731/how-disable-copy-cut-select-select-all-in-uitextview)
> The easiest way to disable pasteboard operations is to create a subclass of UITextView that overrides the canPerformAction:withSender: method to return NO for actions that you don't want to allow:

{% highlight objc %}
- (BOOL)canPerformAction:(SEL)action withSender:(id)sender {
    if (action == @selector(paste:))
        return NO;
    return [super canPerformAction:action withSender:sender];
}
{% endhighlight %}
