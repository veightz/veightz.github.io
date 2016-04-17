---
layout: post
title: description & debugDescription
date: 2015-07-08
---

有时候我们会在代码中直接输出对象，如：

```objc
NSLog(@"%@", objc);
```

原生类的效果还不错，但是自定义的类给出的信息可能只有类型名和内存地址了。

系统有接口让我们对其进行定制。

看了下头文件，在`NSObject.h`可以发现：
```objc
@protocol NSObject

// ...

@property (readonly, copy) NSString *description;
@optional
@property (readonly, copy) NSString *debugDescription;

@end
```

协议`NSObject`(不是类， 是协议) ，声明了`description`，`debugDescription`这两个属性，
ObjectiveC 的两个 Root Class，`NSObject`和`NSProxy`都实现了description，当代码逻辑中直接打印对象的时候，就是调用`description`。
而在`lldb`调试的时候，会默认优先调用`debugDescription`，当`debugDescription`没有实现的时候，会去调用`description`。

由于`debugDescription`是`optional`, `NSObject`并没有实现它，倒是`NSProxy`进行了实现，不过日常开发的时候，用的都是`NSObject`的子类。

测试用例：
```objc
- (NSString *)description {
    return NSStringFromSelector(_cmd);
}

- (NSString *)debugDescription {
    return NSStringFromSelector(_cmd);
}
```

# QuickLook

不过说起来，调试的时候，除了打印文本内容，还可以实现自定义类的`QuickLook`，只要实现`debugQuickLookObject`就行了。
```objc
- (id)debugQuickLookObject
{
    // allocate the return object for the data you wish to represent
    //   Note: "NSImage" is used here arbitrarily for illustration purposes.
    NSImage *_quickLookImage = [...]

    // code that draws a representation of the variable state
    // ...

    // return the object
    return _quickLookImage;
}
```

> [More Details about Enabling Quick Look for Custom Types]( https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/CustomClassDisplay_in_QuickLook/CH01-quick_look_for_custom_objects/CH01-quick_look_for_custom_objects.html)
