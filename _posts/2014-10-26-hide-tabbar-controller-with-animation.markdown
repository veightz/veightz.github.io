---
layout: post
title: Hide Tabbar Controller with Animation
date: 2014-10-26 16:19:10
categories:
---

We always hide some bars animated, like navigation bar:

```objc
[self.navigationController setNavigationBarHidden:YES animated:YES];
```

However, when wo want to hide tab bar, we find we do not have a API to hide it with animation.

```objc
[self.tabBarController.tabBar setHidden:YES];
```

Finally, I get a wonderful way to solve it on `stavkoverflow`.

```objc
- (void)setTabBarVisible:(BOOL)visible animated:(BOOL)animated {

    // bail if the current state matches the desired state
    if ([self tabBarIsVisible] == visible) return;

    // get a frame calculation ready
    CGRect frame = self.tabBarController.tabBar.frame;
    CGFloat height = frame.size.height;
    CGFloat offsetY = (visible)? -height : height;

    // zero duration means no animation
    CGFloat duration = (animated)? 0.3 : 0.0;

    [UIView animateWithDuration:duration animations:^{
        self.tabBarController.tabBar.frame = CGRectOffset(frame, 0, offsetY);
    }];
}

// know the current state
- (BOOL)tabBarIsVisible {
    return self.tabBarController.tabBar.frame.origin.y < CGRectGetMaxY(self.view.frame);
}
```
> http://stackoverflow.com/questions/20935228/how-to-hide-tab-bar-with-animation-in-ios
