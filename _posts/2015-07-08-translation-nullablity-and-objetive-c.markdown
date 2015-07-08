---
layout: post
title: 「译」可选性与ObjectiveC
date: 2015-07-08
---

Swift 能与 ObjectiveC 无缝地交互是一件非常爽的事情，不论是现存的，用 ObjectiveC 写的第三方框架，还是是自己的ObjectiveC 代码。
然而在 Swift 中，可选引用和非可选引用有很大的差别，比如，`NSView` 和 `NSView?` ，在 ObjectiveC 中这两种类型都呈现为 `NSView *` 。
因为 Swift 的编译器不能明确 `NSView *` 是否为可选引用，所以它会以可选值隐式拆包的形式：`NSView!` 在 Swift 中出现。


在Xcode 之前的版本中，一些 Apple 的框架已经被特意处理，让 API 呈现合适的可选属性。
Xcode6.3 提供了一种新的 ObjectiveC 的语言特性来应对这个问题：`nullability annotations`(可选性标注)。

# 核心：`__nullable` 和 `__nonnull`

这个特性的核心，是提供了两个新的类型标注(`type annotations`)：`__nonnull` 和 `__nullable`。
可能你已经猜到了，一个 `__nullable` 指针是允许赋值为 `nil` 和 `NULL` ，而`__nonnull` 指针是不允许这么赋值的。
你敢这么写，编译器就敢让你编译失败。

{% highlight objc %}
@interface AAPLList : NSObject <NSCoding, NSCopying>
// ...
- (AAPLListItem * __nullable)itemWithName:(NSString * __nonnull)name;
@property (copy, readonly) NSArray * __nonnull allItems;
// ...
@end

// --------------

[self.list itemWithName:nil]; // warning!
{% endhighlight %}

一般来说，你能使用 `const` 这个 C 关键字的地方，你就能用 `__nullable` 和 `__nonnull`，
当然啦，必须是给指针类型用。
不过呢，在大部分的情况下，有一种更方便美观的写法，
就是紧接着左括号，直接使用没有下划线的形式，`nullable` 和 `nonnull`。
当然啦，也要是个对象或者 block 的指针。

{% highlight objc %}
- (nullable AAPLListItem *)itemWithName:(nonnull NSString *)name;
- (NSInteger)indexOfItem:(nonnull AAPLListItem *)item;
{% endhighlight %}

对于属性来说，你也能在属性的类型列表中使用这种没有下划线的形式。

{% highlight objc %}
@property (copy, nullable) NSString *name;
@property (copy, readonly, nonnull) NSArray *allItems;
{% endhighlight %}

这种无下划线的形式的确是有下划线的形式好用，但是你需要在每个类型前面加一遍关键字。
为了让工作少点体力活，让你的代码更加清爽，你一定会喜欢用区域性标注(`Audited Regions`)。

# 区域性标注(`Audited Regions`)

为了更方便的配置上新的特性，你可以在你的 ObjectiveC 头文件中，指定某一个区域的代码默认使用`nonnull`。
准确来说，这个区域内的普通指针，都会默认赋予 `nonnull` 属性，
所以我们之前的例子可以简化得更加简单。

{% highlight objc %}
NS_ASSUME_NONNULL_BEGIN
@interface AAPLList : NSObject <NSCoding, NSCopying>
// ...
- (nullable AAPLListItem *)itemWithName:(NSString *)name;
- (NSInteger)indexOfItem:(AAPLListItem *)item;

@property (copy, nullable) NSString *name;
@property (copy, readonly) NSArray *allItems;
// ...
@end
NS_ASSUME_NONNULL_END

// --------------

self.list.name = nil;   // okay

AAPLListItem *matchingItem = [self.list itemWithName:nil];  // warning!
{% endhighlight %}

出于安全性考虑，以下的情况是例外的：

- `typedef` 类型通常不会固定的可空性，因为他们的可选性会基于上下文改变。
因此，`typedef` 类型不会默认假定为 `nonnull`，即便在标注区域内。

- 像 `id *` 之类的比较复杂的指针，必须被显示地标注。
比如说，一个指向一个可为空的引用的，不可为空的指针，
需要使用 `__nullable id * __nonnull`。

- `NSError **` 是被经常被用于返回错误，所以它总会被假定为指向可空引用的可空指针。
关于这个问题，你可以在[错误处理开发指南](http://developer.apple.com/go/?id=error-handling-cocoa)中了解更多。

# 兼容性

你的 ObjectiveC 框架中已经存在的代码该如何应对？就这么改变类型安全吗？放心，安全的。

- 框架中已经被编译的代码将会继续工作，也就是说，ABI 并未改变。
这也意味着在运行时现存的代码不会获得错误的 nil 传递。

- 框架中现存的代码会在编译时，新的 Swift 编译器会对当前不安全行为的使用进行一些附加的警告。

- `nonnull` 并不影响优化。尤其是运行时你依旧能检查被标注为 `nonnull` 的参数是否其实为 `nil`。
这在做向后兼容时，可能显得格外重要。

# 回到 Swift

现在我们已经可以在 ObjectiveC 的头文件里，让我们从 Swift 中使用它。

标注前：

{% highlight swift %}
class AAPLList : NSObject, NSCoding, NSCopying {
	// ...
	func itemWithName(name: String!) -> AAPLListItem!
	func indexOfItem(item: AAPLListItem!) -> Int

	@NSCopying var name: String! { get set }
	@NSCopying var allItems: [AnyObject]! { get }
	// ...
}
{% endhighlight %}

标注后：

{% highlight swift %}
class AAPLList : NSObject, NSCoding, NSCopying {
	// ...
	func itemWithName(name: String) -> AAPLListItem?
	func indexOfItem(item: AAPLListItem) -> Int

	@NSCopying var name: String? { get set }
	@NSCopying var allItems: [AnyObject] { get }
	// ...
}
{% endhighlight %}

现在 Swift 代码更加干净了。这微妙的改变，会让你使用你的框架更加的愉悦。

对 C 和 ObjectiveC 的可空性标注从 Xcode6.3 开始提供。
更多信息，请看[ Xcode 6.3 Release Notes](http://developer.apple.com/go/?id=xcode-6.3-beta-release-notes)。

> 查看原文: [Nullability and Objective-C](https://developer.apple.com/swift/blog/?id=25)
