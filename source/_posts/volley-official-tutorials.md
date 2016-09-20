---
title: Volley官方教程翻译
date: 2016-09-20 15:18:16
tags: android volley
---

一直以来我都认为官方教程和源码是快速学习技术与深入理解的最佳途径。
所以我们先粗略翻译一下官方教程，然后在通过源码分析一下具体的实习细节。

# 使用Volley来完成网络数据请求

> 原文连接：https://developer.android.com/training/volley/index.html

Volley是一个HTTP库，它似的Android上的应用程序的网络访问变得更加方便、快速。你可以通过[AOSP](https://android.googlesource.com/platform/frameworks/volley)(Android Open Source Project)获取其源码。
Volley包含在Android源码之中，具体路径为：android源码/frameworks/volley

Volley有一下优点：
- 自动调度网络请求
- 多并发网络连接
- 具有标准缓存一致性的，透明的，内存、硬盘响应缓存
- 支持请求优先级
- 取消请求接口。你可以取消一个单独的请求，你也可以通过划定范围取消多个请求。
- 方便定制。比如说：重试和退避请求。
- 强大的排序功能，使得你能够很容易的使用从网络异步获取的数据填充你的UI。
- 调试与追踪工具。

Volley尤其擅长用来填充UI的RPC调用操作，比如获取一组结构化的搜索结果。它很容易与其他协议集成，而且已经实现了常用的、原始的String，Image与Json协议请求。Volley提供了许多在日常开发中所需要的网络层的功能，避免我们去重复造轮子，专注于业务逻辑代码的实现。

Volley不是用于大文件下载或者数据流的操作，因为在处理响应的时候，Volley把整个响应都保存在内存中。如果要操作大文件，请考虑使用DownloadManager。

