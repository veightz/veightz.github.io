---
layout: post
title: "Could not find developer disk image"
date: 2015-10-22 21:02:00
categories:
---

今天真机调试时，显示我的设备 unavailable， 可昨天还是好的。

查了下相关信息，在真机调试时，会检查有没有对应版本时候支持真机调试的版本

>
/Applications/Xcode/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport

要求前两位版本匹配。看了下现在本地（此时Xcode7.0），只有9.0的包。

所以，早上升了 iOS9 9.1 的我，该去升级 Xcode 了。
