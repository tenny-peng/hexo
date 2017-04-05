---
title: 多台电脑hexo部署到github的坑
date: 2017-04-05 10:54:05
tags: hexo
categories: hexo+github搭建博客
---

之前在家里搭建了博客，成功部署到github上，并将hexo目录也上传至github/hexo仓库保存。
后来到公司想把它们down下来，方便两边修改同步。

## 第一个坑是hexo发布博客到github

同样进行了一系列的node安装，hexo安装等，并且在hexo博客目录下down下了guthub/hexo的资源，本地启动，没问题。
但是当我发布想发布到yourname.github.io上时，问题来了，它居然把我的**整个hexo博客目录**扔到了yourname.github.io上，不是说好的只发布**.deploy_git**下的内容呢！！

于是我就茫然了啊，我去查看hexo下的_config.yml文件，
```
deploy:
  type: git
  repo: git@github.com:tenny-peng/tenny-peng.github.io.git
  branch: master
```
没错啊，是这个地址啊。

后来一想也不对，就算这里错了也不对，不是目标地址错了，而是发布的内容错了。

网上查到了这篇博客: [hexo部署到github遇到的坑](http://www.jianshu.com/p/67c57c70f275)，最后说删除hexo目录下的.git文件，然后我就试了试，重新发布，然后又报错了。。这里想截图可是命令行找不到了，大概就是说没有指定repository，然后我点开**.deploy_git**文件夹，突然想到在家里**.deploy_git**文件夹下面是有.git的，而且还是我自己指定的。

哈哈，瞬间好像知道了，打开git bash，切换到hexo/.deploy_git，执行
```
git init
```
再绑定远程仓库
```
git remote add origin git@github.com:tenny-peng/tenny-peng.github.io.git
```
回到cmd命令行
```
hexo d
```
搞定了，成功提交了正确的博客内容。

## 第二个坑是hexo目录与github/hexo同步

然后再把我的hexo目录和github/hexo同步，刚才把hexo目录下的.git删了。好吧，重新建回来。
git bash切换到hexo根目录
```
git init

git remote add origin git@github.com:tenny-peng/hexo.git
```
执行pull指令
```
$ git pull origin master
From github.com:tenny-peng/hexo
 * branch            master     -> FETCH_HEAD
error: The following untracked working tree files would be overwritten by merge:
        .npmignore
        _config.yml
        db.json
        node_modules/.bin/JSONStream
        node_modules/.bin/JSONStream.cmd
        node_modules/.bin/acorn
        node_modules/.bin/acorn.cmd
        ...
```
这里说一下，第一个坑中hexo目录下的内容是我直接从github/hexo上down下来复制过来的，然后本地又进行过hexo生成和发布操作，错误具体原因不太清除(知道的童鞋欢迎指正)，网上查到解决办法是先清理
```
$ git clean -f -d
Skipping repository .deploy_git/
Removing .npmignore
Removing _config.yml
Removing db.json
Removing node_modules/
Removing package.json
Removing public/
Removing scaffolds/
Removing source/
Removing themes/
```
清理后hexo目录下只剩.deploy_git和.git目录，再拉取就可以了
```
$ git pull origin master
From github.com:tenny-peng/hexo
 * branch            master     -> FETCH_HEAD
Checking out files: 100% (7651/7651), done.
```
所以下次可以先同步好hexo文件夹，这样第二个坑应该就不会出现了。
