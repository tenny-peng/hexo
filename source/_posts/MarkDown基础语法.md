---
title: Markdown基础语法
date: 2017-04-03 10:21:15
tags: [markdown, 语法]
categories: markdown
---
# 标准mrakdown语法

## 加粗和斜体

### 字体加粗以两个*或两个_开头结尾
```
__重要的事情说三遍！！！__   **重要的事情说三遍！！！**
```
效果如下：

**重要的事情说三遍！！！**

### 字体倾斜以一个*或_开头结尾
```
_一段斜体文字_   *一段斜体文字*
```
效果如下：

 *一段斜体文字*

## 链接和邮件

### 链接：
```
bla bla bla [example](http://url.com/ "Title")
这是我们常用的网站: [百度一下](www.baidu.com "百度一下")
```
效果如下：

这是我们常用的网站: [baidu](www.baidu.com "百度一下")

也可以定义一个id来对应链接地址
```
bla bla bla [example][id]. Then, anywhere else in the doc, define the link:
[id]: http://example.com/  "Title"

这是我们常用的网站: [baidu][baiduId]
[baiduId]: www.baidu.com "百度一下"
```
效果如下：

这是我的[博客][blog]，欢迎访问。
[blog]:https://tenny-peng.github.io "tenny's blog"

### 邮件：
```
An email <example@example.com> link.
这是我的邮箱<mpengtaoqi@163.com>。
```
效果如下：

这是我的邮箱<mpengtaoqi@163.com>。


## 图片

行内引用 (标题可选):
```
![alt text](/path/img.jpg "Title")
这里出现一个引用的图片![图片替代文字](https://tenny-peng.github.io/images/avatar.jpg "偷得浮生半日闲")
```
效果如下：

这里出现一个引用的图片![图片替代文字](https://tenny-peng.github.io/images/avatar.jpg "偷得浮生半日闲")



使用id引用图片链接：
```
![alt text][id]
[id]: /url/to/img.jpg "Title"

使用id引用图片链接![图片替代文字][img_id]
[img_id](https://tenny-peng.github.io/images/avatar.jpg "偷得浮生半日闲")
```
效果如下：

使用id引用图片链接![图片替代文字][imgId]
[imgId]: https://tenny-peng.github.io/images/avatar.jpg "偷得浮生半日闲"


## 标题

### 底线形式:
```
标题 1
========

标题 2
--------
```
效果如下：

标题 1
========

标题 2
--------

### #模式 (末尾的#可选):
```
# 标题 1 #

## 标题 2 ##

###### 标题 6
```
效果如下：

# 标题 1 #

## 标题 2 ##

###### 标题 6

## 列表
### 有序的, 不带段落:
```
1.  Git
2.  Hexo
3.  MarkDown
```
效果如下：

1.  Git
2.  Hexo
3.  MarkDown

### 无序的, 带段落:
```
*   一个条目.

    巴拉巴拉拉，这里是段落文字.

*   其他条目
```
效果如下：

*   一个条目.

    巴拉巴拉巴拉，这里是段落文字.

*   其他条目

### 你可以嵌套使用它们：
```
*   Work
    * java
*   Blog
    1.  github
    2.  atom
        * markdown
    3. hexo
*   learn
```
效果如下：

*   Work
    * java
*   Blog
    1.  github
    2.  atom
        * markdown
    3. hexo
*   learn

## 区块引用
```
> 类似邮件的引用方式
> 在断好的行前加上`>`

> > 也可以嵌套使用

> #### 引用标题
>
> * 也可以是一个列表
> * 等等
```
效果如下：

> 类似邮件的引用方式
> 在断好的行前加上`>`

> > 也可以嵌套使用

> #### 引用标题
>
> * 也可以是一个列表
> * 等等

### 内联代码
```
`<some code>` 使用引号和反引号标记行内代码片段。

如果要在代码区段内插入反引号，可以用多个反引号来开启和结束代码区段。

例如 `` `this` ``.
```
效果如下：

`<some code>` 使用引号和反引号标记行内代码片段。

如果要在代码区段内插入反引号，可以用多个反引号来开启和结束代码区段。

例如 `` `this` ``.

### 代码块
在句段的行首插入1个tab 或 4个空格，则表示代码块。
```
这是一段普通的文字。

  这一段代码块。
```

## 分隔线
用三个以上的星号或减号或底线来建立一个水平分隔线。

行内不能有其他东西，但你可以在星号或是减号中间插入空格。
```
---

* * *

- - - -
```
效果如下：

---

* * *

- - - -

## 换行
在一行的结尾处加上2个或2个以上的空格
```
Roses are red,   
Violets are blue.
```
效果如下：

Roses are red,   
Violets are blue.
