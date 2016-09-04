---
title: Git 服务器搭建
date: 2016-05-02 22:31:44
tags: git tools
---

# 安装Git
 省略...

# 搭建服务器
 首先创建一个Git服务器的文件夹，方便同一管理

``` bash
mkdir /home/kzz/GitRepository
```
 使用--bare创建Git服务器

``` bash
cd /home/kzz/GitRepository
git init --bare Blog.git
```
从这条指令有两点需要说明一下：
--bare参数。bare的意思是'裸的'，如果初始化的时候添加了这个参数意味着它将不保存源文件的拷贝，只存在.git文件夹下的文件，这种更适合于服务器。

# 拉去服务器仓库

在本地拉取服务器仓库的方法
``` bash
cd ~/workspace
git clone kzz@ip:/home/kzz/GitRepository/Blog.git
```
