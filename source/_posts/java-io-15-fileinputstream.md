---
title: 15 - Java IO FileInputStream
date: 2015-01-15 17:36:55
tags: [java, java io]
categories: java io
---

> [原文链接](http://tutorials.jenkov.com/java-io/fileinputstream.html) 作者: Jakob Jenkov 译者: 李璟(jlee381344197@gmail.com)

FileInputStream可以以字节流的形式读取文件内容。FileInputStream是InputStream的子类，这意味着你可以把FileInputStream当做InputStream使用(FileInputStream与InputStream的行为类似)。

这是一个FileInputStream的例子：

``` java
InputStream input = new FileInputStream("c:\\data\\input-text.txt");
int data = input.read();while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = input.read();
}
input.close();
```

请注意，为了清晰，这里忽略了必要的异常处理。想了解更多异常处理的信息，请参考[Java IO异常处理]()。

FileInputStream的read()方法返回读取到的包含一个字节内容的int变量(译者注：0~255)。如果read()方法返回-1，意味着程序已经读到了流的末尾，此时流内已经没有多余的数据可供读取了，你可以关闭流。-1是一个int类型，不是byte类型，这是不一样的。

FileInputStream也有其他的构造函数，允许你通过不同的方式读取文件。请参考官方文档查阅更多信息。

其中一个FileInputStream构造函数取一个File对象替代String对象作为参数。这里是一个使用该构造函数的例子：

``` java
File file = new File("c:\\data\\input-text.txt");
InputStream input = new FileInputStream(file);
```

至于你该采用参数是String对象还是File对象的构造函数，取决于你当前是否已经拥有一个File对象，也取决于你是否要在打开FileOutputStream之前通过File对象执行某些检查(比如检查文件是否存在)。

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: FileInputStream](http://ifeve.com/java-io-fileinputstream/)