---
layout: post
title: "如何在 Block 中不加持 self 的情况下调用父类的实现"
date: 2015-10-27 00:08:00
categories:
---
# 问题出现

在使用 Block 的时候，大家都会留意在 Block 中使用 `self` 的问题，因为容易造成循环引用。
如果在自己的一个 block 属性中直接使用了 `self`，编译器娘还会给你报个 warning 提示。

> Capturing 'self' strongly this block likely to lead to retain cycle

这个问题大家都知道怎么解决

```objc
__weak typeof(self) weakSelf = self;
```

或者用 `@weakify(self)` 和 `@strongify(self)` 去处理。

现在假设某个需求，需要你在这个会导致引用循环的 block 了调用父类的实现，怎么破？
比如说我们要进行 `[super foo]` 这样的调用。

在 ObjectiveC 中，一般的对象方法调用都会转换成 `objc_msgSend()` 这样的形式，
不过 `super` 就比较特别了，它并不像 `self` 是个变量。
`super` 调用方法，都是会转换成 `objc_msgSendSuper(self, ...)` 的形式，
于是问题来了，`self` 被持有住了。

这一步转换引入的 `self`，不是上层开发时能控制的。
就算用了 `weakSelf` 也无济于事，编译器还是会用 `self`，而不是你的 `weakSelf`。
`super` 调用某方法的本质，是 `self` 调用某方法的父类实现。
也是解决这个问题的入口。

# 解决思路

## 准备

我们先创建两个类 Father 和 Child， Child 是 Father 的子类。
它们都有一个 foo 方法，输出自己的类名。
防止引入其他 Runtime 的特性造成验证干扰，没有使用 `NSStringFromClass()` 去获取类名。

```objc
@interface Father : NSObject
- (void)foo;
@end

@implementation Father
- (void)foo {
    NSLog(@"Father");
}
@end
```

```objc
@interface Child : Father
@property (nonatomic, copy) void (^callback)();
- (void)configThenInvokeBlock;
@end

@implementation Child
- (void)configThenInvokeBlock {
    __weak typeof(self) weakSelf = self;
    [self setCallback:^{
        // 关键区域
    }];
    self.callback();
}
- (void)foo {
    NSLog(@"Child");
}
- (void)super_foo {
    [super foo];
}
- (void)dealloc {
    NSLog(@"Child instance released.");
}
```

## 方法一：包装父类调用

这是第一种方法，比较土，但是也是最方便和清楚的方式。
创建了一个子类的方法，它的实现是去调用父类实现，在 block 中调用这个创建的子类方法。

```objc
- (void)configThenInvokeBlock {
    __weak typeof(self) weakSelf = self;
    [self setCallback:^{
        /**
          *  1. 包装父类方法
          */
        [weakSelf super_foo];
    }];
    self.callback();
}

- (void)super_foo {
    [super foo];
}
```
要是需要调用的父类实现比较多，那就是体力活了+_=

## 方法二：让 weakSelf 执行父类方法实现

```objc
- (void)configThenInvokeBlock {
    __weak typeof(self) weakSelf = self;
    [self setCallback:^{
        /**
         *  2. 运行时获取父类实现 再使用 weakSelf 调用  
         *  -- 注意需要 @import ObjectiveC;
         */
        Method fatherMethod = class_getInstanceMethod([weakSelf superclass], @selector(foo));
        void (*method_invokeCasted)(id, Method) = (void *)method_invoke;
        method_invokeCasted(weakSelf, fatherMethod);
    }];
    self.callback();
}
```

通过`class_getInstanceMethod`拿到父类的方法的实现，然后用 `method_invoke` 去调用执行。
不过这里要做一下函数的类型转换，以免 LLVM 吐槽 `too many arguments`。

## 方法三：通过 weakSelf 构造 objc\_super，调用 objc_msgSendSuper

```objc
- (void)configThenInvokeBlock {
    __weak typeof(self) weakSelf = self;
    [self setCallback:^{
        /**
          *  3. 构造 objc_super 再使用 objc_msgSendSuper 调用
          *  -- 注意需要 @import ObjectiveC;
          */
        struct objc_super superStruct = {weakSelf, [weakSelf superclass]};
        void (*objc_msgSendSuperCasted)(struct objc_super *, SEL) = (void *)objc_msgSendSuper;
        objc_msgSendSuperCasted(&superStruct, @selector(foo));
    }];
    self.callback();
}
```

这里的思路是利用 `objc_msgSendSuper` 去实现调用父类实现。
先使用 `weakSelf` 去构造一个 `objc_super` 类型的结构体，
然后做一个函数的类型转换，获得`objc_msgSendSuperCasted`，
以此直接调用父类实现。

# 题外话

1. 如果是 block 最多被试图执行一次的话，还是手动破环吧，比较有安全感。
2. 还需要研究下确认下 `@weakify(self)` 和 `@strongify(self)` 这两个宏(hei)定(mo)义(fa)在这个问题下的影响。

方便大家测试， 贴一段混合的 configThenInvokeBlock 全家桶。

```objc
- (void)configThenInvokeBlock {
    __weak typeof(self) weakSelf = self;
    [self setCallback:^{

        /**
         *  经典引用循环错误
         */
//        [self foo];
        /**
         *  常见处理手段
         */
//        [weakSelf foo];


        /**
         *  使用 super 实际上还是持有 self，会造成引用循环
         */
//        [super foo];

        /**
         *  下面是三种处理办法
         */

        /**
         *  1. 包装父类方法
         */
//        [weakSelf super_foo];

        /**
         *  2. 运行时获取父类实现 再使用 weakSelf 调用
         */
//        Method fatherMethod = class_getInstanceMethod([weakSelf superclass], @selector(foo));
//        void (*method_invokeCasted)(id, Method) = (void *)method_invoke;
//        method_invokeCasted(weakSelf, fatherMethod);

        /**
         *  3. 构造 objc_super 再使用 objc_msgSendSuper 调用
         */
//        struct objc_super superStruct = {weakSelf, [weakSelf superclass]};
//        void (*objc_msgSendSuperCasted)(struct objc_super *, SEL) = (void *)objc_msgSendSuper;
//        objc_msgSendSuperCasted(&superStruct, @selector(foo));
    }];
    self.callback();
}
```
