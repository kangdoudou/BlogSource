---
title: 20 - Java IO PipedOutputStream
date: 2015-01-20 17:36:55
tags: [java, java io]
categories: java io
---

> [原文链接](http://tutorials.jenkov.com/java-io/pipedoutputstream.html) 作者: Jakob Jenkov 译者: 李璟(jlee381344197@gmail.com)

PipedOutputStream可以往管道里写入读取字节流数据，代码如下：

``` java
OutputStream output = new PipedOutputStream(pipedInputStream);
while(moreData) {
    int data = getMoreData();
    output.write(data);
}
output.close();
```

请注意，为了清晰，这里忽略了必要的异常处理。想了解更多异常处理的信息，请参考[Java IO异常处理]()。

PipedOutputStream的write()方法取一个包含了待写入字节的int类型变量作为参数进行写入。

# Java IO管道
一个PipedOutputStream总是需要与一个PipedInputStream相关联。当这两种流联系起来时，它们就形成了一条管道。要想更多地了解Java IO中的管道，请参考Java IO管道。

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: PipedOutputStream](http://ifeve.com/java-io-pipedoutputstream/)