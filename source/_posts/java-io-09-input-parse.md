---
title: 09 - Java IO 输入解析
date: 2015-01-09 17:36:55
tags: [java, java io]
categories: java io
---

>  [原文链接](http://tutorials.jenkov.com/java-io/input-parsing.html) 作者: Jakob Jenkov 译者: KangZongZhan

Java IO API中的一些类是被设计用来解析输入的。 这些类有：

- PusbackInputStream
- PusbackReader
- StreamTokenizer
- PushbackReader
- LineNumberReader

这篇文章的目的不是详细教你如何解析数据，而是带你快速认识一下关于数据解析方面都有那几个类。

当你需要解析数据的时候，最后你都会使用上面列表中的一个或几个类来实现自己的一个类。过去我在写Butterfly Container Script的时候就这样做过。我的解析器的核心部分使用的是`PushbackInputStream`，因为有些时候我需要读取之前的一两个字符来决定当前字符的含义。

我的一篇文章介绍了我一个实际的实例，使用'PushbackReader'来[替换流、字节字符数组、文件中的字符串](http://tutorials.jenkov.com/java-howto/replace-strings-in-streams-arrays-files.html)。这个实例创建了一个'TokenReplacingReader'，它能够替换从底层的Reader中读取的格式如${记号}的记号。用户使用`TokenReplacingReader`的时候根本就不知道发生了字符串的替换。
