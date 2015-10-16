---
layout: post
title: imageNamed && imageWithContentsOfFile
date: 2015-08-03 22:40:22
---

最近把之前资源瘦身的分支合并了下, 毕竟剁手宝的包已经逼近 100MB 了. 图片资源迁移到 bundle 后, 好像不能在里面新建抽象的 Group 了.
所以我把资源在 bundle 中分了文件夹, 并做了个接口.

{% highlight objc %}
if (!image) {
    NSString *imageFullPath = [[NSBundle mainBundle] pathForResource:imageName ofType:typeString inDirectory:directoryString];
    if (cached) {
        image = [UIImage imageNamed:imageFullPath];
    } else {
        image = [UIImage imageWithContentsOfFile:imageFullPath];
    }
}
{% endhighlight %}

然后发现, 刚 cache 为 YES, 执行 `image = [UIImage imageNamed:imageFullPath]` 的时候,
image 始终初始化失败, 返回 nil.
于是在控制台试了几把.

{% highlight sh %}
(lldb) po imageName
my_head_stroke_72

(lldb) po imageFullPath
/Users/veightz/Library/Developer/CoreSimulator/Devices/3F04333C-8B21-43FD-9759-CCDFCE6BD5A1/data/Applications/6E367A3D-6D31-4973-8933-1ED6F1FF6BE3/TBMainClient.app/TBMyTaoBao.bundle/Default/my_head_stroke_72.png

(lldb) po [UIImage imageNamed:imageFullPath]
 nil
(lldb) po [UIImage imageNamed:imageName]
<UIImage: 0x7b085c10>

(lldb) po [UIImage imageWithContentsOfFile:imageFullPath]
<UIImage: 0x78f128e0>

(lldb) po [UIImage imageWithContentsOfFile:imageName]
 nil
 {% endhighlight %}

 {% highlight objc %}
 if (!image) {
     NSString *imageFullPath = [[NSBundle mainBundle] pathForResource:imageName ofType:typeString inDirectory:directoryString];
     if (cached) {
         image = [UIImage imageNamed:imageName];
     } else {
         image = [UIImage imageWithContentsOfFile:imageFullPath];
     }
 }
 {% endhighlight %}

嗯。。就是这样。。
