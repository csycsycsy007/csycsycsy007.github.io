---
title: category 是怎么从 runtime 层面实现的
date: 2020-02-21 08:00:00 +0800
categories: [iOS]
tags: [OC]
---

## 背景

Category 是 Objective-C 中的一种机制，允许你在不修改原始类的情况下，为已有类添加方法。Category 的实现依赖于 Objective-C 的运行时（runtime）系统。下面从 runtime 层面详细解释 Category 是如何实现的。

## 编译时的处理

当你在代码中定义一个 Category 时，编译器会将这个 Category 编译成一个与原始类相关联的数据结构。这个数据结构包括 Category 的名称、它扩展的类的名称，以及 Category 中定义的方法列表（`objc_method_list`）。

## 运行时的加载

当程序启动时，Objective-C 的 runtime 系统会加载所有的类和它们的 Category。Category 的方法并不会直接编译进类的结构中，而是在程序加载时动态地合并到类的结构中。

## 方法列表的合并

在程序启动或者 Category 所在的动态库加载时，runtime 会将 Category 的方法列表合并到目标类的方法列表中。具体步骤如下：

1. **查找类的定义**：runtime 通过类名找到 Category 所要扩展的类。
2. **方法列表合并**：runtime 将 Category 中的方法添加到该类的 `objc_method_list` 中。如果 Category 中定义的方法与类中已有的方法同名，Category 中的方法会覆盖原类中的方法。

## 方法调用

当你调用一个方法时，Objective-C runtime 会在类的方法列表中查找方法的实现。由于 Category 的方法已经被合并到类的方法列表中，runtime 能够直接找到并执行这些方法。

## 数据结构

在 runtime 层面，Category 的数据结构大致如下：

- `objc_class`：代表类的结构体，包含类的方法列表、属性列表等。
- `objc_category`：代表 Category 的结构体，包含 Category 的名称、它所属的类的名称，以及方法列表、属性列表等。
- `objc_method_list`：包含类或 Category 的所有方法。

## 动态库中的处理

如果 Category 是在动态库（如 .dylib）中定义的，那么在该动态库加载时，Category 的方法会动态添加到原始类中。这使得 Category 可以在运行时动态地修改类的行为。

## 关键点总结

- Category 是通过将方法列表动态合并到类的结构中实现的。
- Category 中的方法在运行时加载时被添加到类的方法列表中。
- 如果 Category 中的方法与原类的方法重名，则会覆盖原方法。
- 这种机制允许在不修改类源代码的情况下，动态地扩展或修改类的行为。

## 代码示例

为了更好地理解 Category 是如何在 Objective-C 的 runtime 层面实现的，下面通过代码示例以及 runtime 函数展示 Category 的实现过程。

### 示例代码：定义一个 Category

首先，我们定义一个简单的 Category 来扩展 `NSString` 类，并添加一个新方法：

```objective-c
// NSString+Reverse.h
#import <Foundation/Foundation.h>

@interface NSString (Reverse)
- (NSString *)reverseString;
@end

// NSString+Reverse.m
#import "NSString+Reverse.h"

@implementation NSString (Reverse)
- (NSString *)reverseString {
    NSUInteger len = [self length];
    NSMutableString *reversedString = [NSMutableString stringWithCapacity:len];

    for (NSInteger i = len - 1; i >= 0; i--) {
        [reversedString appendString:[self substringWithRange:NSMakeRange(i, 1)]];
    }

    return reversedString;
}
@end
```

## Runtime 实现解析

Category 在编译阶段被转化为 runtime 数据结构，并在加载时合并到原始类的 `method_list` 中。我们可以使用 Objective-C runtime 的相关函数来验证 Category 的方法确实被添加到了类中。

### 动态添加方法示例

在 `main.m` 中，通过 runtime API 来打印 `NSString` 的所有方法，包含通过 Category 添加的方法：

```objective-c
#import <Foundation/Foundation.h>
#import <objc/runtime.h>
#import "NSString+Reverse.h"

void printMethodsForClass(Class cls) {
    unsigned int methodCount = 0;
    Method *methods = class_copyMethodList(cls, &methodCount);

    for (unsigned int i = 0; i < methodCount; i++) {
        SEL methodSelector = method_getName(methods[i]);
        const char *methodName = sel_getName(methodSelector);
        NSLog(@"Method #%d: %s", i, methodName);
    }

    free(methods);
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // 创建一个NSString对象
        NSString *string = @"Objective-C";

        // 调用Category方法
        NSString *reversedString = [string reverseString];
        NSLog(@"Reversed String: %@", reversedString);

        // 打印NSString的所有方法
        NSLog(@"Methods in NSString:");
        printMethodsForClass([NSString class]);
    }
    return 0;
}
```

### 运行结果

在运行上述代码后，除了 `NSString` 类原本的方法外，还会打印出 `reverseString` 方法，这证明了 Category 的方法已经成功地被添加到类中。

### 使用 `class_copyMethodList` 检查方法列表

通过使用 `class_copyMethodList` 函数，我们能够获取并打印类的所有方法，包括通过 Category 动态添加的方法。

```objective-c
unsigned int count;
Method *methods = class_copyMethodList([NSString class], &count);

for (unsigned int i = 0; i < count; i++) {
    Method method = methods[i];
    SEL selector = method_getName(method);
    const char *name = sel_getName(selector);
    NSLog(@"Method %u: %s", i, name);
}

free(methods);
```

### Category 数据结构

在 runtime 中，Category 对应的结构体定义如下：

```objective-c
struct objc_category {
    const char *name;             // Category 名称
    Class cls;                    // 被扩展的类
    struct objc_method_list *instanceMethods; // 实例方法列表
    struct objc_method_list *classMethods;    // 类方法列表
    struct objc_protocol_list *protocols;     // Category 实现的协议
};
```

### 结论

通过以上代码示例和 runtime 函数调用，你可以看到 Category 是如何动态地在运行时将方法添加到类中。Objective-C runtime 通过将 Category 的方法列表合并到类的 method_list 中，实现在不修改原类的情况下扩展类的功能。这种机制使得 Category 成为一个非常强大的工具，尤其在需要为现有类添加功能而不想创建子类的情况下。
