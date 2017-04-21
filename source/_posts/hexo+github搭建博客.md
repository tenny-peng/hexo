---
layout: title
title: hexo+github搭建博客
date: 2017-04-21 14:07:53
tags: [hexo, github]
categories: hexo+github博客
---

Hexo是一款基于Node.js的静态博客框架，配合github可以搭建属于自己的博客。

# 搭建环境

## 安装Node.js
Hexo需要node.js支持，可以到[node.js中文网](http://nodejs.cn/download/)下载适合自己系统的安装包。安装也比较简单，一路next下去就可以了。

安装完后win + r 输入cmd回车，打开命令行界面，分别输入node -v 和nmp -v，看到类似如下结果就说明安装成功了。
```
>node -v
v6.10.1
>nmp -v
3.10.10
```

## 安装Hexo
在合适的地方建立一个文件夹，用于安装hexo框架和存放你的博客。我的文件夹是D:\devsoft\hexo。

命令行切换到hexo目录
```
C:\Users\tenny>d:

D:\>cd devsoft

D:\devsoft>cd hexo

D:\devsoft\hexo>
```
输入如下命令安装hexo到当前目录
```
npm install hexo-cli -g
```
命令行显示一系列安装详情，等待片刻，完成后，继续输入
```
npm install hexo --save
```
又会看到一堆信息，完成后，输入hexo -v检查下，看到类似如下信息，说明安装成功了。
```
D:\devsoft\hexo>hexo -v
hexo: 3.2.2
hexo-cli: 1.0.2
os: Windows_NT 10.0.14393 win32 x64
http_parser: 2.7.0
node: 6.10.1
v8: 5.1.281.95
uv: 1.9.1
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 58.2
modules: 48
openssl: 1.0.2k
```

## 配置Hexo
命令行还在hexo根目录，输入
```
hexo init
```
继续输入
```
npm -install
```
npm会自动安装需要的组件。之后输入
```
npm install hexo-deployer-git --save
```
hexo扩展，用于将博客发布到github上。

# 体验博客

## 本地博客
继续输入hexo g生成文件
```
D:\devsoft\hexo>hexo g
INFO  Start processing
INFO  Files loaded in 2.28 s
INFO  Generated: search.xml
INFO  Generated: sitemap.xml
INFO  Generated: atom.xml
INFO  Generated: index.html
INFO  Generated: categories/index.html
INFO  Generated: about/index.html
INFO  Generated: tags/index.html
INFO  Generated: archives/index.html
INFO  Generated: favicon.ico
INFO  Generated: archives/2017/04/index.html
......   //省略的文件信息
```
再输入hexo s启动服务
```   
D:\devsoft\hexo>hexo s
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
在浏览器中输入http://localhost:4000/ 就可以看到博客首页了。

要停止服务，在命令行按Ctrl + C。

## 建立博客仓库
进入https://github.com/ 登录自己的账号，新建一个仓库，命名为yourname.github.io(这个就是你博客的访问地址，一定要这种格式，否则无效)。例如我的tenny-peng.github.io。

关于安装git和github可以参考我的[Git简单教程](../../03/Git简单教程/index.html)，这里就略过了。

建立好自己的博客仓库(yourname.github.io)后，打开hexo根目录下的_config.yml，找到Deployment，修改成如下内容
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master
```
例如我的
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:tenny-peng/tenny-peng.github.io.git
  branch: master
```

## 编写博客
hexo根目录下执行
```
hexo new title "test"
```
然后在D:\devsoft\hexo\source\_posts下就能看到test.md文件了。

.md文件是用MarkDown语法写的，关于MarkDown语法，可以参考我的[MarkDown基础语法](../../03/MarkDown基础语法/index.html)。  
MarkDown文件编辑器推荐用Atom，Atom是Github专门为程序员推出的一个跨平台文本编辑器。可以到https://atom.io/ 下载Atom，也可以找寻其他自己喜欢的MarkDown编辑器。

## 部署博客
文章编辑完后，使用命令生成，部署
```
hexo g      //生成静态文件
hexo d      //部署到github上
```
也可以直接执行以下命令，相当于上面两条命令一起执行
```
hexo d -g     //部署前先生成
```
部署完成后，访问https://yourname.github.io (例如我的https://tenny-peng.github.io) ，就可以看到生成的文章。
