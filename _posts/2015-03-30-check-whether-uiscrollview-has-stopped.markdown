---
layout: post
title: "获得滚动视图停止的正确姿势"
date: 2015-03-30 01:06:31
categories:
---

由于项目需要, 做了一个滚动视图, 不过需要它停留在一些固定的离散位置, 所以要在用户变动滚动视图偏移后, 进行一次位置的校正. 所以要在合适的实际进行校正函数的调用.

# Objective C 下的 hacks

在[栈溢出](http://stackoverflow.com/questions/993280/how-to-detect-when-a-uiscrollview-has-finished-scrolling)上很快找到之前的通解.
```objc
- (void)scrollViewDidScroll:(UIScrollView *)sender {
  [NSObject cancelPreviousPerformRequestsWithTarget:self];
  //ensure that the end of scroll is fired.
  [self performSelector:@selector(scrollViewDidEndScrollingAnimation:) withObject:nil afterDelay:0.3];
  ...
}

- (void)scrollViewDidEndScrollingAnimation:(UIScrollView *)scrollView {
  [NSObject cancelPreviousPerformRequestsWithTarget:self];
  // ...
}
```

# Swift 的悲剧

翻文档的时候发现悲了个剧

	Sending Messages
	performSelector:withObject:afterDelay:  Not available in Swift
	performSelector:withObject:afterDelay:inModes: Not available in Swift
	performSelectorOnMainThread:withObject:waitUntilDone: Not available in Swift
	performSelectorOnMainThread:withObject:waitUntilDone:modes: Not available in Swift
	performSelector:onThread:withObject:waitUntilDone: Not available in Swift
	performSelector:onThread:withObject:waitUntilDone:modes: Not available in Swift
	performSelectorInBackground:withObject: Not available in Swift
	cancelPreviousPerformRequestsWithTarget(_:)
	Cancels perform requests previously registered with the performSelector:withObject:afterDelay: instance method.


Swift 下没法使用相关接口..

# 新的思路

静下了仔细想一下,  用户所带来的停止, 其实是两种情况

1. drag 完后, 有一个decelerated motion 的过程, did end decelerating 后, 就是停止
  这个其实很好处理, 在`scrollViewDidEndDecelerating(scrollView: UIScrollView)`的回调中就是捕获.

2. drag 完后,  没有产生惯性, 直接停下来了, 所以我们要在did end  dragging 后就做一次判断,
	所以我们可以在`scrollViewDidEndDragging(scrollView: UIScrollView, willDecelerate decelerate: Bool)`中, 对参数`decelerate`进行判断.

```swift
// MARK: UIScrollViewDelegate
extension YourViewControllerOrViewClass: UIScrollViewDelegate {
  // case 1:
  func scrollViewDidEndDecelerating(scrollView: UIScrollView) {
    adjustScrollViewPostion(scrollView)
  }
  // case 2:
  func scrollViewDidEndDragging(scrollView: UIScrollView, willDecelerate decelerate: Bool) {
    if !decelerate {
      adjustScrollViewPostion(scrollView)
    }
  }

  // MARK: Handle method
  func adjustScrollViewPostion(scrollView: UIScrollView) {
    // ...
  }
}
```
