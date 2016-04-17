---
layout: post
title: "使用 Dispatch Group"
date: 2015-03-31 00:35:51
---

最近在业务逻辑中实践了一下大中枢派发。简单的串行异步队列通常不能满足需求, 比如你要在多个异步任务完成后才进行某个操作.

# Group 的创建

```objc
dispatch_group_t group = dispatch_group_create();
```
这没什么好说的.

# 任务的添加

## 方法一

```objc
dispatch_group_async(group, dispatch_get_main_queue(), ^{
  NSLog(@"foo");
});
dispatch_group_async(group, dispatch_get_main_queue(), ^{
  NSLog(@"bar");
});
```
适用于一些本地逻辑, 比如同时需要查多张表, 全部查完再 reload data 之类的.

## 方法二
有时候业务的需要是在多个网络请求响应后做一些处理, 而一般我们都会把 HTTP 相关的接口直接封装成异步的, 用上面的方法就会导致变成异步事件中的异步事件.

这时候我们需要下面两个接口:

```objc
dispatch_group_enter(group);
dispatch_group_leave(group);
```

需要注意的是, 这两个接口一定要确保被调用的次数是一致的, 在处理响应结果的逻辑中, 要保证不管哪一种逻辑, 都要考虑到这个一致问题.没猜错的话, 这是信号量实现的, 不一致的话就永远不回调了...

# 完成后的回调
也有两个接口

```objc
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
  NSLog(@"finished.");
});
```

## 阻塞方式
这个接口会阻塞当前队列, 第二个参数表示等待时间.

```objc
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
```

## 异步方式
这个就是非阻塞了, 通过 block 传入回调函数, 之后的代码继续执行.

```objc
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
  NSLog(@"finished.");
});
```
