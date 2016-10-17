---
title: 06 - Java IO 字节和字符数组
date: 2015-01-06 17:36:55
tags: [java, java io]
categories: java io
---

> [原文链接](http://tutorials.jenkov.com/java-io/arrays.html)  作者： Jakob Jenkov   译者：homesick

内容列表

- 从InputStream或者Reader中读入数组
- 从OutputStream或者Writer中写数组

在java中常用字节和字符数组在应用中临时存储数据。而这些数组又是通常的数据读取来源或者写入目的地。如果你需要在程序运行时需要大量读取文件里的内容，那么你也可以把一个文件加载到数组中。当然你可以通过直接指定索引来读取这些数组。但如果设计成为从InputStream或者Reader，而不是从数组中读取某些数据的话，你会用什么组件呢？

# 从 InputStream 或 Reader中读取数组

用ByteArrayInputStream或者CharArrayReader封装字节或者字符数组从数组中读取数据。通过这种方式字节和字符就可以以数组的形式读出了。

样例如下：

``` java
byte[] bytes = new byte[1024];
//把数据写入字节数组...
InputStream input = new ByteArrayInputStream(bytes);
//读取第一个字节
int data = input.read(); 
while(data != -1) {
//操作数据

//读下一个字节
data = input.read();
}
```

以同样的方式也可以用于读取字符数组，只要把字符数组封装在CharArrayReader上就行了。

# 通过 OutputStream 或者 Writer写数组

同样，也可以把数据写到ByteArrayOutputStream或者CharArrayWriter中。你只需要创建ByteArrayOutputStream或者CharArrayWriter，把数据写入，就像写其它的流一样。当所有的数据都写进去了以后，只要调用toByteArray()或者toCharArray，所有写入的数据就会以数组的形式返回。

样例如下：

``` java
OutputStream output = new ByteArrayOutputStream();
output.write("This text is converted to bytes".toBytes("UTF-8"));
byte[] bytes = output.toByteArray();
```

写字符数组也和此例子类似。只要把字符数组封装在CharArrayWriter上就可以了。

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: 字节和字符数组](http://ifeve.com/java-io-array/)