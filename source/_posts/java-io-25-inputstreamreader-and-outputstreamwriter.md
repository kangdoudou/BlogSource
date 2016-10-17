---
title: 25 - Java IO InputStreamReader和OutputStreamWriter
date: 2015-01-25 17:36:55
tags: [java, java io]
categories: java io
---

> 作者: Jakob Jenkov 译者: 李璟(jlee381344197@gmail.com)

本章节将简要介绍InputStreamReader和OutputStreamWriter。细心的读者可能会发现，在之前的文章中，IO中的类要么以Stream结尾，要么以Reader或者Writer结尾，那这两个同时以字节流和字符流的类名后缀结尾的类是什么用途呢？简单来说，这两个类把字节流转换成字符流，中间做了数据的转换，类似适配器模式的思想。


# InputStreamReader
[原文链接](http://tutorials.jenkov.com/java-io/inputstreamreader.html)

InputStreamReader会包含一个InputStream，从而可以将该输入字节流转换成字符流，代码例子：

``` java
InputStream inputStream = new FileInputStream("c:\\data\\input.txt");
Reader reader = new InputStreamReader(inputStream);
int data = reader.read();
while(data != -1){
    char theChar = (char) data;
    data = reader.read();
}
reader.close();
```

注意：为了清晰，代码忽略了一些必要的异常处理。想了解更多异常处理的信息，请参考Java IO异常处理。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注：0~65535)。代码如下：

``` java
int data = reader.read();
```

你可以把返回的int值转换成char变量，就像这样：

``` java
char aChar = (char) data; //译者注：这里不会造成数据丢失，因为返回的int类型变量data只有低16位有数据，高16位没有数据
```

如果方法返回-1，表明Reader中已经没有剩余可读取字符，此时可以关闭Reader。-1是一个int类型，不是byte或者char类型，这是不一样的。

InputStreamReader同样拥有其他可选的构造函数，能够让你指定将底层字节流解释成何种编码的字符流。例子如下：

``` java
InputStream inputStream = new FileInputStream("c:\\data\\input.txt");
Reader reader = new InputStreamReader(inputStream, "UTF-8");
```

注意构造函数的第二个参数，此时该InputStreamReader会将输入的字节流转换成UTF8字符流。

# OutputStreamWriter
[原文链接](http://tutorials.jenkov.com/java-io/outputstreamwriter.html)

OutputStreamWriter会包含一个OutputStream，从而可以将该输出字节流转换成字符流，代码如下：

``` java
OutputStream outputStream = new FileOutputStream("c:\\data\\output.txt");
Writer writer = new OutputStreamWriter(outputStream);
writer.write("Hello World");
writer.close();
```

OutputStreamWriter同样拥有将输出字节流转换成指定编码的字符流的构造函数。

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: InputStreamReader和OutputStreamWriter](http://ifeve.com/java-io-inputstreamreader%E5%92%8Coutputstreamwriter/)
