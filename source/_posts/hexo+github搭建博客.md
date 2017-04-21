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

# 使用主题
主题可以使我们的博客更加个性化，更加美观，等等。
这里我使用了NexT主题，其他主题配置可参考其说明，下面以NexT为例。

## 安装NexT
Hexo 安装主题的方式非常简单，只需要将主题文件拷贝至站点目录的 themes 目录下， 然后修改下配置文件即可。

如果你熟悉Git，建议你使用克隆最新版本的方式，之后的更新可以通过git pull来快速更新，而不用再次下载压缩包替换。这里我们使用git。

命令行切换到hexo根目录，执行
```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

## 启用主题
找到hexo根目录下的站点配置文件_config.yml，修改theme
```
theme: next
```

## 设定Scheme
Scheme 是 NexT 提供的一种特性，借助于 Scheme，NexT 为你提供多种不同的外观。同时，几乎所有的配置都可以 在 Scheme 之间共用。目前 NexT 支持三种 Scheme，他们是：
* Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
* Mist - Muse 的紧凑版本，整洁有序的单栏外观
* Pisces - 双栏 Scheme，小家碧玉似的清新

找到主题next目录下的_config.yml(**注意**：不是hexo根目录下的配置文件，根目录下的是全局博客配置，这个是针对某个主题的配置)，设定自己喜欢的Scheme，使用的去掉#，不使用的注释#。
```
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
#scheme: Muse
#scheme: Mist
scheme: Pisces
```

## 站点设置
编辑站点配置文件，设置博客标题，作者，语言等，更多配置可自行查询。
```
# Site
title: Tenny's Blog
author: Tenny Peng
language: zh-Hans     //简体中文
```

## 菜单配置
编辑主题配置文件，设置首页分类标签等目录，更多配置可自行查询。
```
menu:
  home: /
  categories: /categories
  about: /about
  archives: /archives
  tags: /tags
```
这里设定的目录都必须手动创建在hexo/source目录下，否则发布到github上是找不到的。

## 头像设置
编辑主题配置文件，修改avatar(如没有可新建)
```
avatar: /images/avatar.jpg
```
这里的图片需要放在主题目录下(themes/next/source/images/avatar.jpg)，而不是站点目录。

## 网站图标
编辑主图配置文件，修改favicon
```
favicon: /favicon.ico
```
然后将favicon.ico放在hexo/source目录下即可。

<font size="5">以上基本就完成了个人博客的搭建，更多信息可参考：</font>

[史上最详细的Hexo博客搭建图文教程](https://xuanwo.org/2015/03/26/hexo-intor/)

[hexo官网](https://hexo.io/)

[next文档](http://theme-next.iissnan.com/getting-started.html)
