
---
layout: post
title: git 不区分大小写???
date: 2020-10-30 18:00:00 +0800
category: debug
thumbnail: /style/image/debug.png
icon: code
---

* content
{:toc}

# git 忽略大小写


## 踩坑
> 开发环境
> IntelliJ IDEA Ultimate 2020.2
> git, Source Tree, Windows 10, centos

在使用IDEA修改了文件名file大小写,使用Source Tree提交后,
线上分支出现了File,file两个文件,
但是在windows/mac 拉取分支时,只显示一个文件

在centos上拉取时, 两个文件都拉下来了

奇奇怪怪的问题经历增加了...

## 原因
git 默认不区分文件名大小写
但是Linux上配置了文件名大小写,所以它能把两个文件都拉下来

而不区分大小写的情况下,拉取到的是后更改的文件
所以从本地看项目中没有大小写不同的两个文件...

## 解决办法
git config 中可以指定对文件名大小写的区分

在项目目录下执行以下git 命令
```Shell
git config core.ignorecase false
```
也可以修改 git 全局的配置

```Shell
git config --global core.ignorecase false
```