---
title: 28 - Java IO 字符流的Piped和CharArray
date: 2015-01-28 17:36:55
tags: [java, java io]
categories: java io
---

> 作者: Jakob Jenkov 译者: 李璟(jlee381344197@gmail.com)

本章节将简要介绍管道与字符数组相关的reader和writer，主要涉及PipedReader、PipedWriter、CharArrayReader、CharArrayWriter。

# PipedReader
[原文链接](http://tutorials.jenkov.com/java-io/pipedreader.html)

PipedReader能够从管道中读取字符流。与PipedInputStream类似，不同的是PipedReader读取的是字符而非字节。换句话说，PipedReader用于读取管道中的文本。代码如下：

``` java
Reader reader = new PipedReader(pipedWriter);
int data = reader.read();
while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = reader.read();
}
reader.close();
```

注意：为了清晰，代码忽略了一些必要的异常处理。想了解更多异常处理的信息，请参考[Java IO异常处理](/2015/01/12/java-io-12-exception/)。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注：0~65535)。如果方法返回-1，表明PipedReader中已经没有剩余可读取字符，此时可以关闭PipedReader。-1是一个int类型，不是byte或者char类型，这是不一样的。

正如你所看到的例子那样，一个PipedReader需要与一个PipedWriter相关联，当这两种流联系起来时，就形成了一条管道。要想更多地了解Java IO中的管道，请参考[Java IO管道](/2015/01/04/java-io-04-pipes/)。

# PipedWriter
[原文链接](http://tutorials.jenkov.com/java-io/pipedwriter.html)

PipedWriter能够往管道中写入字符流。与PipedOutputStream类似，不同的是PipedWriter处理的是字符而非字节，PipedWriter用于写入文本数据。代码如下：

``` java
PipedWriter writer = new PipedWriter(pipedReader);
while(moreData()) {
    int data = getMoreData();
    writer.write(data);
}
writer.close();
```

PipedWriter的write()方法取一个包含了待写入字节的int类型变量作为参数进行写入，同时也有采用字符串、字符数组作为参数的write()方法。

# CharArrayReader
[原文链接](http://tutorials.jenkov.com/java-io/chararrayreader.html)

CharArrayReader能够让你从字符数组中读取字符流。代码如下：

``` java
char[] chars = ... //get char array from somewhere.
Reader reader = new CharArrayReader(chars);
int data = reader.read();
while(data != -1) {
    //do something with data
    data = reader.read();
}
reader.close();
```

如果数据的存储媒介是字符数组，CharArrayReader可以很方便的读取到你想要的数据。CharArrayReader会包含一个字符数组，然后将字符数组转换成字符流。(译者注：CharArrayReader有2个构造函数，一个是CharArrayReader(char[] buf)，将整个字符数组创建成一个字符流。另外一个是CharArrayReader(char[] buf, int offset, int length)，把buf从offset开始，length个字符创建成一个字符流。更多细节请参考[Java官方文档](http://docs.oracle.com/javase/7/docs/api/))

# CharArrayWriter
[原文链接](http://tutorials.jenkov.com/java-io/chararraywriter.html)

CharArrayWriter能够把字符写入到字符输出流writer中，并且能够将写入的字符转换成字符数组。代码如下：

``` java
CharArrayWriter writer = new CharArrayWriter();
//write characters to writer.
char[] chars = writer.toCharArray();
```

当你需要以字符数组的形式访问写入到writer中的字符流数据时，CharArrayWriter是个不错的选择。

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: 字符流的Piped和CharArray](http://ifeve.com/java-io-%E5%AD%97%E7%AC%A6%E6%B5%81%E7%9A%84piped%E5%92%8Cchararray/)