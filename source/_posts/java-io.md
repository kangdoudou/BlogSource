---
title: 01-Java IO教程
date: 2015-01-01 17:19:10
tags: [java, java io]
categories: java io
---

> [原文链接](http://tutorials.jenkov.com/java-io/index.html) 作者：Jakob Jenkov  译者：Connor (cronnorc@gmail.com) ，李璟  校对：方腾飞

Java IO 是一套Java用来读写数据（输入和输出）的API。大部分程序都要处理一些输入，并由输入产生一些输出。Java为此提供了java.io包。

如果你浏览下java.io包，会对其中各样的类选择感到迷惑。这些类的作用都是什么？对于某个任务该选择哪个类？怎样创建你自己的类做插件？这个手册的目的就是给你介绍这些类是如何组织的，以及怎样使用他们，因此你就不会疑惑需要时怎样选取合适的类，或者是否有一个满足你需求的类已经存在了。


# Java.io 包的范围
java.io 包并没有涵盖所有输入输出类型。例如，并不包含GUI或者网页上的输入输出，这些输入和输出在其它地方都涉及，比如Swing工程中的JFC (Java Foundation Classes) 类，或者J2EE里的Servlet和HTTP包。

Java.io 包主要涉及文件，网络数据流，内存缓冲等的输入输出。

# 更多的Java IO工具，提示等
这个手册也被称为” [Java How To’s and Utilities](http://tutorials.jenkov.com/java-howto/index.html) ”，包含一些Java IO的工具，例如替换流数据中的字符串，使用缓冲来反复处理流数据。

# 此Java IO 手册的范围
这个手册开始部分会给你一个Java IO API 工作的概览，以及你该怎样使用这些他们，接着会介绍包括所有Java IO API 的核心类。

这个手册不只是一个API的列表，这样的列表你可以从Sun公司的官方Java文档获得。事实上，每篇文档都是对一个类的简要介绍，设计它的目的以及一些实用的例子。换句话说，这些内容你在Sun公司的官方文档上是找不到的。

（本文是第一篇，如果你有兴趣翻译剩下的文章，请在回复中领取文章，翻译后，可以讲译文直接邮箱给我，或者直接发布在并发网上，你也可以加入我们试译者QQ群领取其他文章翻译，369468545）


1. [Java IO 教程]()
2. [Java IO 概述]()
3. [Java IO: 文件]()
4. [Java IO: 管道]()
5. [Java IO: 网络]()
6. [Java IO: 字节和字符数组]()
7. [Java IO: System.in, System.out, and System.error]()
8. [Java IO: 流]()
9. [Java IO: Input Parsing（暂无翻译，处理中）]()
10. [Java IO: Readers and Writers]()
11. [Java IO: 并发IO]()
12. [Java IO: 异常处理]()
13. [Java IO: InputStream]()
14. [Java IO: OutputStream]()
15. [Java IO: FileInputStream]()
16. [Java IO: FileOutputStream]()
17. [Java IO: RandomAccessFile]()
18. [Java IO: File]()
19. [Java IO: PipedInputStream]()
20. [Java IO: PipedOutputStream]()
21. [Java IO: 字节流的ByteArray和Filter]()
22. [Java IO: 字节流的Buffered和Data]()
23. [Java IO: 序列化与ObjectInputStream、ObjectOutputStream]()
24. [Java IO: Reader和Writer]()
25. [Java IO: InputStreamReader和OutputStreamWriter]()
26. [Java IO: FileReader和FileWriter]()
27. [Java IO: 字符流的Buffered和Filter]()
28. [Java IO: 字符流的Piped和CharArray]()
29. [Java IO: 其他字节流(上)]()
30. [Java IO: 其他字符流(下)]()

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO教程](http://ifeve.com/java-io/)