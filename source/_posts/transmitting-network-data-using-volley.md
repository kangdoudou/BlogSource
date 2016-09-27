---
title: Volley官方教程翻译（一）：使用Volley来完成网络数据请求
date: 2016-09-27 22:20:50
tags: android volley
---

> 原文连接：https://developer.android.com/training/volley/index.html

Volley是一个HTTP库，它使得Android应用程序的网络访问变得更加方便、快速。你可以通过[AOSP](https://android.googlesource.com/platform/frameworks/volley)(Android Open Source Project)获取其源码。
Volley包含在Android源码之中，具体路径为：android源码/frameworks/volley

Volley有以下优点：

- 自动调度网络请求
- 多并发网络连接
- 具有标准缓存一致性的，透明的，内存、硬盘响应缓存
- 支持请求优先级
- 取消请求接口。你可以单独取消一个请求，你也可以通过划定范围取消多个请求
- 方便定制。比如说：重试和退避请求
- 强大的排序功能，使得你能够很容易的使用从网络异步获取的数据填充你的UI
- 调试与追踪工具

Volley尤其擅长用来填充UI的RPC调用操作，比如获取一组结构化的搜索结果。它很容易与其他协议集成，而且已经实现了常用的、原始的String，Image与Json协议请求。Volley提供了许多在日常开发中所需要的网络层的功能，避免我们去重复造轮子，专注于业务逻辑代码的实现。

Volley不是用于大文件下载或者数据流的操作，因为在处理响应的时候，Volley把整个响应都保存在内存中。如果要操作大文件，请考虑使用DownloadManager。

Volley库源码位于AOSP仓库的frameworks/volley目录下，它的核心内容包括请求调度的流水线以及“toolbox”目录内的一些常用工具类的实现。将Volley应用与你的项目的最简单的方法就是克隆Volley仓库，然后将其设置为类库项目。

1.  执行如下指令，将Volley项目克隆到本地。当然需要梯子了，没梯子的请自行百度下载。
`git clone https://android.googlesource.com/platform/frameworks/volley`
2. 将下载的源码作为类库项目导入到你的项目中，具体做法[传送门](https://developer.android.com/studio/projects/android-library.html)。