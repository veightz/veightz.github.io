---
layout: post
title: 一个很无聊的Alfredworkflow
date: 2014-09-26 00:14:57
categories:
---

打开Github的速度太慢了,大家都懂的.每次在一个repo里想搜其他内容的时候,点回主页再搜索感觉好麻烦啊=-=.(求小技巧打脸!).然后写了个小workflow偷懒=-=.

> [点击下载](http://vtypecho.qiniudn.com/Github.alfredworkflow)

# 使用
![使用图例](http://vtypecho.qiniudn.com/workflow_github.jpg)

# 实现
随便在github搜个内容,就能看到请求的整个URL:

{% highlight sh%}
https://github.com/search?utf8=**✓**&q=afnetwork
{% endhighlight %}

于是在Alfred Workflow的配置里添加了:

{% highlight sh%}
https://github.com/search?utf8=✓&q={query}
{% endhighlight %}

可是发现报错了,看了下, 是 **✓** 过不了=-=, 不太懂Web, 不知道怎么绕过去,希望高手解答下=-=.
于是我粗暴的写了:

{% highlight sh%}
https://github.com/search?q={query}
{% endhighlight %}

说起来还真是暗搓搓的=-=
