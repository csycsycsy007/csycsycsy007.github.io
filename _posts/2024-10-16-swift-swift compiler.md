---
title: Swift Compiler
date: 2024-10-16 08:00:00 +0800
categories: [iOS]
tags: [swift]
---

## **是什么？**
swift 编译器，把 swift 代码转化为高效、优化的可执行程序。

## **我能做什么？**
能通过 xcode 的 Build Settings 改一些相关配置, 比如：Compilation Mode

## **我能得到什么？**
能做些优化

### **优化编译时间**

- **增量编译模式**

使用增量编译模式可以显著减少每次修改后重新编译的时间。确保你在Debug配置中使用增量编译模式（默认设置）。

在 Build Settings 中，找到 Swift Compiler - Code Generation 下的 Compilation Mode。
选择 Incremental 模式。
减少文件数量和大小
大的源文件和大量文件会增加编译时间。将一个大的源文件拆分成更小的文件，可以减少编译时间。

- **优化依赖关系**

减少不必要的依赖，尽可能使用协议和抽象层来解耦模块之间的依赖关系。

- **使用预编译头文件（PCH）**

可以将常用头文件放到预编译头文件中，减少重复编译代码的时间。虽然Swift不直接支持预编译头文件，但你可以通过创建包含Objective-C桥接头文件的PCH来优化：

创建一个预编译头文件（.pch）。
在 Build Settings 中找到 Prefix Header，并添加你的预编译头文件路径。

- **关闭不必要的警告**

虽然保持代码清洁很重要，但某些情况下可以关闭不影响关键功能的警告来优化编译时间：

在 Build Settings 中找到 Swift Compiler - Warnings and Errors。

调整或关闭不需要的警告。

### **使用编译器标志进行优化**
通过 Other Swift Flags 配置项，你可以传递特定编译器标志来优化编译行为：

**Whole Module Optimization**
启用整个模块优化有助于发布（Release）构建，可以生成更快、更高效的代码：

```sh
-whole-module-optimization
```

添加在 Other Swift Flags 中：

在 Build Settings 中找到 Other Swift Flags。

添加 -whole-module-optimization 标志。

### **跟踪编译时间和调试信息**
编译时间和性能分析对于大型项目尤为重要。你可以使用Xcode内部的编译时间日志功能进行跟踪和分析。

**启用编译时间日志**

打开终端并运行以下命令，启用编译时间日志：

```sh
defaults write com.apple.dt.Xcode ShowBuildOperationDuration YES
```

**使用Swift编译器标志记录编译时间**

在 Build Settings 中找到 Other Swift Flags 配置项，并添加以下标志来记录详细编译时间：

```sh
-Xfrontend -debug-time-compilation
```
**查看编译时间报告**

编译完成后，编译时间会显示在Xcode的构建日志中。你可以分析这些时间来找到需要优化的部分。


