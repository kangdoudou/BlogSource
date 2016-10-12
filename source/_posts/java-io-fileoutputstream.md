---
title: 16 - Java IO FileOutputStream
date: 2015-01-16 17:36:55
tags: [java, java io]
categories: java io
---

> [原文链接](http://tutorials.jenkov.com/java-io/fileoutputstream.html) 作者: Jakob Jenkov 译者: 李璟(jlee381344197@gmail.com)

FileOutputStream可以往文件里写入字节流，它是OutputStream的子类，所以你可以像使用OutputStream那样使用FileOutputStream。

这是一个FileOutputStream的例子：

``` java
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt");
while(moreData) {
    int data = getMoreData();
    output.write(data);
}
output.close();
```

请注意，为了清晰，这里忽略了必要的异常处理。想了解更多异常处理的信息，请参考[Java IO异常处理]()。

FileOutputStream的write()方法取一个包含了待写入字节(译者注：低8位数据)的int变量作为参数进行写入。

FileOutputStream也有其他的构造函数，允许你通过不同的方式写入文件。请参考[官方文档]()查阅更多信息。

# 文件内容的覆盖Override VS追加Appending
当你创建了一个指向已存在文件的FileOutputStream，你可以选择覆盖整个文件，或者在文件末尾追加内容。通过使用不同的构造函数可以实现不同的目的。

其中一个构造函数取文件名作为参数，会覆盖任何此文件名指向的文件。

``` java
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt");
```

另外一个构造函数取2个参数：文件名和一个布尔值，布尔值表明你是否需要覆盖文件。这是构造函数的例子：

``` java
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt", true); //appends to file
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt", false); //overwrites file
```

# 写入字节数组
既然FileOutputStream是OutputStream的子类，所以你也可以往FileOutputStream中写入字节数组，而不需要每次都只写入一个字节。可以参考我的OutputStream教程查阅更多关于写入字节数组的信息。

# flush()
当你往FileOutputStream里写数据的时候，这些数据有可能会缓存在内存中。在之后的某个时间，比如，每次都只有X份数据可写，或者FileOutputStream关闭的时候，才会真正地写入磁盘。当FileOutputStream没被关闭，而你又想确保写入到FileOutputStream中的数据写入到磁盘中，可以调用flush()方法，该方法可以保证所有写入到FileOutputStream的数据全部写入到磁盘中。

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: FileOutputStream](http://ifeve.com/java-io-fileoutputstream/)