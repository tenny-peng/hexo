---
title: Hello World!
date: 2017-04-08 17:34:50
tags: [java]
categories: Java学习记录
---

# 开始动手，Hello World！！
是时候编写你自己的第一个Java应用了。

## 下载JDK
你可以到这里[下载](http://www.oracle.com/technetwork/java/javase/downloads/index.html)JDK并安装。

注意：下载的是**JDK**，而*不是*JRE。JRE是java运行环境，用于运行java程序；JDK是java开发工具包，用于开发java程序，其中包含了JRE，所以我们下载JDK就好。

安装完成后，win + r 输入cmd，打开命令行窗口，输入"java -version"，看到类似如下结果就说明安装成功了。
```
C:\Users\Administrator>java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

## 编辑工具
这里用windows自带的记事本就可以。

## 创建源文件
在D盘新建一个目录myapplication(你也可以自己选择其他盘符及目录)，新建一个记事本文档，将下面代码粘贴或手动输入到文本里：
```
/**
 * The HelloWorldApp class implements an application that
 * simply prints "Hello World!" to standard output.
 */
class HelloWorldApp {
    public static void main(String[] args) {
        System.out.println("Hello World!"); // Display the string.
    }
}
```
注意：Java严格**区分大小写**，HelloWorldApp不等于helloWorldapp。

将该文件保存为HelloWorldApp.java。

## 编译
win + r，输入cmd，打开命令行窗口，输入
```
cd d:\myapplication
```
切换到HelloWorldApp.java文件所在目录。

切换目录使用如下命令：
```
C:\>D:    //切换到D盘根目录

D:\>cd myapplication    //切换到当前目录下的myapplication目录

D:\myapplication>     //完成切换。。
```

继续输入:
```
javac HelloWorldApp.java
```
如果没有任何信息，应该就编译成功了，查看myapplication目录，发现多出一个HelloWorldApp.class文件，这个就是字节码文件。  
如果出现错误提示，请检查文件名和文件内容是否和上述一致。

## 运行
输入java HelloWorldApp运行程序。
```
D:\myapplication>java HelloWorldApp
Hello World!
```
看到打印出了"Hello World!"说明我们得程序运行成功了。  
如果提示错误，请检查文件名和文件内容是否和上述一致。

至此，第一个java应用程序就完成了。
