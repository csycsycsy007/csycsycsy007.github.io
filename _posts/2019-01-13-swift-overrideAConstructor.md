---
title: swift 重写构造函数有什么意义？ 为什么有的需要有的不需要构造？
date: 2019-01-15 08:00:00 +0800
categories: [iOS]
tags: [swift]
---

#### 1. 重写构造函数的意义

- **定制初始化过程**：

  - 当子类需要在实例创建时执行与父类不同的初始化步骤时，重写构造函数是必要的。例如，子类可能需要初始化更多的属性或进行特定的配置。

- **扩展父类的初始化逻辑**：

  - 通过重写构造函数，子类可以在保留父类初始化逻辑的基础上，添加新的初始化步骤。这通常需要在重写的构造函数中调用 `super.init` 以确保父类的初始化操作被执行。

- **避免使用不适合的父类构造函数**：
  - 在某些情况下，父类的某些构造函数可能不适用于子类。通过重写这些构造函数，并根据需要标记为 `required` 或将其设为 `unavailable`，可以避免使用不适合的初始化方式。

#### 2. 为什么有时需要重写，有时不需要

- **自动继承父类的构造函数**：

  - 如果子类没有定义新的存储属性，并且父类的构造函数已经适用于子类的初始化需求，则子类可以直接继承父类的构造函数，而无需重写。

- **为新属性或特定需求提供初始化**：

  - 如果子类定义了新的存储属性（且这些属性没有默认值），必须在构造函数中初始化它们，这就需要重写构造函数。
  - 如果子类需要执行比父类更多的初始化步骤，也需要重写构造函数来满足这些需求。

- **便利构造函数（Convenience Initializer）**：
  - 子类可能不需要重写父类的指定构造函数（designated initializer），但可以添加便利构造函数，以提供更便捷的实例化方式。便利构造函数通常会调用指定构造函数，从而简化对象创建的过程。

#### 3. 示例说明

```swift
class Animal {
    var name: String

    init(name: String) {
        self.name = name
    }
}

class Dog: Animal {
    var breed: String

    // 重写父类的构造函数以初始化新属性
    override init(name: String) {
        self.breed = "Unknown"
        super.init(name: name)
    }

    // 添加新的构造函数以初始化所有属性
    init(name: String, breed: String) {
        self.breed = breed
        super.init(name: name)
    }
}

let dog1 = Dog(name: "狗蛋")
print(dog1.name)  // 输出: 狗蛋
print(dog1.breed) // 输出: Unknown

let dog2 = Dog(name: "狗蛋", breed: "哈士奇")
print(dog2.name)  // 输出: 狗蛋
print(dog2.breed) // 输出: 哈士奇
```
