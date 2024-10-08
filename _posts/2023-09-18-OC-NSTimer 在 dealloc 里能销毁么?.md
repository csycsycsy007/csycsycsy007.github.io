---
title: NSTimer 在 dealloc 里能销毁么?
date: 2023-09-10 08:00:00 +0800
categories: [iOS]
tags: [OC]
---

在 Objective-C 中，`NSTimer` 是一个常用的计时器类，用于定时执行特定的代码。然而，`NSTimer` 的使用也带来了一个潜在的内存管理问题：如果不正确处理，它会导致对象无法被正常销毁，进而造成内存泄漏。这篇文章将深入探讨为什么 `NSTimer` 需要在 `dealloc` 方法中手动销毁，以及如何正确地实现这一点。

## 为什么 `NSTimer` 可能导致内存泄漏？

`NSTimer` 的工作机制决定了它会对其目标对象（通常是 `self`）保持强引用，直到计时器被无效化（invalidate）为止。这意味着，除非你明确告诉 `NSTimer` 停止工作并释放资源，否则它会一直持有目标对象的引用，导致目标对象无法被释放，即使 `dealloc` 方法已经被调用。

想象一下，你有一个视图控制器使用 `NSTimer` 来定期更新 UI。然而，当你关闭这个视图控制器时，计时器仍然在后台悄悄运行，目标对象（视图控制器本身）无法被释放，因为 `NSTimer` 还在引用它。这就是典型的内存泄漏情景。

## 如何在 `dealloc` 方法中手动销毁 `NSTimer`？

为了避免上述问题，最佳实践是在对象的 `dealloc` 方法中手动使 `NSTimer` 无效化。这确保了计时器不再触发任何回调，并释放对目标对象的强引用。

```objective-c
- (void)dealloc {
    [self.timer invalidate]; // 使 NSTimer 无效化
    self.timer = nil; // 释放对 NSTimer 的引用
    // 在 ARC 环境下，系统会自动调用 [super dealloc]
}
```

## 代码解析

`[self.timer invalidate]`: 这一步使 `NSTimer` 停止工作，并且从其运行循环中移除。一旦无效化，计时器将不再触发任何操作，也不会继续保持对目标对象的强引用。
`self.timer = nil`: 这不仅是清理代码的好习惯，更是为了确保对计时器对象的引用被彻底清除，使其能够被 ARC（自动引用计数）正确管理和释放。
NSTimer 的内存管理：细节决定成败
在 ARC 环境下，尽管 `dealloc` 方法不再需要显式调用 `[super dealloc]`，但对 NSTimer 的无效化操作依然必不可少。忽视这一点，很可能导致应用程序中隐秘而顽固的内存泄漏问题。

## 结论

`NSTimer` 的便利性不可否认，但它背后的内存管理细节更值得我们关注。在 `dealloc` 方法中手动使 `NSTimer` 无效化是避免内存泄漏的关键步骤。通过这一简单而必要的操作，我们能够确保目标对象的生命周期得到正确管理，使我们的代码更加健壮和可靠。

记住，每一个 `NSTimer` 的实例都需要我们在使用完毕后，进行适当的清理。无论是 `dealloc` 还是其他生命周期管理方法，这一操作都是 Objective-C 中内存管理不可忽视的一部分。
