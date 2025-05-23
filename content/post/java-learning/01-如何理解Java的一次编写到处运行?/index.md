---
title: "如何理解Java的 “一次编写，到处运行”"
description:
date: "2024-01-20T15:35:28+08:00"
slug: "write-once-run-anywhere"
image: ""
license: false
hidden: false
comments: false
draft: false
tags: ["Java", "JVM"]
categories: ["Java", "JVM"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---

## Write once, run anywhere

`“一次编写，到处运行”`说的是Java语言跨平台的特性，Java的跨平台特性与Java虚拟机的存在密不可分，可在不同的环境中运行。比如说Windows平台和Linux平台都有相应的JDK，安装好JDK后也就有了Java语言的运行环境。其实Java语言本身与其他的编程语言没有特别大的差异，并不是说Java语言可以跨平台，而是在不同的平台都有可以让Java语言运行的环境而已，所以才有了Java一次编写，到处运行这样的效果。

严格的讲，跨平台的语言不止Java一种，但Java是较为成熟的一种。`“一次编写，到处运行”`这种效果跟编译器有关。编程语言的处理需要编译器和解释器。Java虚拟机和DOS类似，相当于一个供程序运行的平台。

程序从源代码到运行的三个阶段：编码——编译——运行——调试。Java在编译阶段则体现了跨平台的特点。编译过程大概是这样的：首先是将Java源代码转化成`.CLASS`文件字节码，这是第一次编译。 `.class` 文件就是可以到处运行的文件。然后Java字节码会被转化为目标机器代码，这是是由JVM来执行的，即Java的第二次编译。

“到处运行”的关键和前提就是JVM。因为在第二次编译中JVM起着关键作用。在可以运行Java虚拟机的地方都内含着一个JVM操作系统。从而使JAVA提供了各种不同平台上的虚拟机制，因此实现了“到处运行”的效果。需要强调的一点是，java并不是编译机制，而是解释机制。Java字节码的设计充分考虑了JIT这一即时编译方式，可以将字节码直接转化成高性能的本地机器码，这同样是虚拟机的一个构成部分。

## Java 是解释运行吗？

***答案是 不是***

### Java 是如何编译的？

1. 编译阶段：
    1. Java源代码（`.java`）通过`javac编译器` **提前编译** 为平台无关的字节码（`.class`）
    2. 这一步是传统的AOT（Ahead-Of-Time, 预先编译，编译在程序运行前完成）编译，生成中间代码而非机器码
2. 运行时阶段：
    1. **解释执行：** JVM（如HotSpot）首先逐行解释执行字节码（`.class`）。
    2. **JIT编译：** （Just in time，即时编译）对于频繁执行的代码（热点代码，方法级代码），JVM触发JIT编译器（如C1/C2）将其编译为本地机器码，大幅提升性能，属于运行时动态优化。
