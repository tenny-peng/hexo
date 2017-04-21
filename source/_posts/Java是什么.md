---
title: Java是什么
date: 2017-04-08 14:30:18
tags: [java]
categories: Java学习记录
---

# Java 技术
Java技术既是一门编程语言，也是一个平台。

## Java编程语言
Java作为一门高级编程语言，具有如下特性：

* 简单
* 跨平台
* 面向对象
* 便携性
* 分布式
* 高性能
* 多线程
* 健壮的
* 动态的
* 安全的

在Java程序中，所有的源文件是以".java"结尾的普通文本文件，java编译器将源文件编译成".class"文件。".class"文件包含的并不是类似"0101011"的机器语言代码，而是称之为字节码(bytecodes)的东东，而这个字节码其实是java虚拟机的机器语言。通过java虚拟机，再将字节码转换为本地机器可识别的代码。

编译图如下：

![java编译图](http://docs.oracle.com/javase/tutorial/figures/getStarted/getStarted-compiler.gif)

因为java虚拟机可运行在不同的平台，所以我们的.class文件也可以运行在微软，Linux,苹果等不同平台。虚拟机的存在使得java实现了跨平台，也就是经常说的"一次编译，到处运行"。

![java跨平台](http://docs.oracle.com/javase/tutorial/figures/getStarted/helloWorld.gif)

## Java平台
平台是供程序运行的硬件或软件环境。我们常说的微软操作系统，Linux系统，苹果操作系统这些都属于平台。大部分的平台是操作系统和基础硬件的合集。而Java平台与大多数平台不同，它是一个纯的软件平台，运行于其他的基于硬件的平台上(例如Windows)。

Java平台包含两部分：
* Java虚拟机
* Java应用程序接口(API)

上面已经提到过java虚拟机，它是java平台的基础，与不同的操作系统对接。

API是内置的许多有用的方法的集合。它将相关的类(class)和接口(interface)组合成库，这个库我们通常称之为包(package)。

![java平台](http://docs.oracle.com/javase/tutorial/figures/getStarted/getStarted-jvm.gif)

作为一个平台独立的环境，Java平台的速度比传统编译(c++直接编译成机器指令，java编译后是字节码文件，还需要虚拟机翻译成机器指令)慢，。但是随着硬件的发展，java编译器和虚拟机的性能已大幅提升，其速度和传统编译已相差无几。
