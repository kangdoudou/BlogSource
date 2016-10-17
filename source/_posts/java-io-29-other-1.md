---
title: 29 - Java IO 其他字节流(上)
date: 2015-01-29 17:36:55
tags: [java, java io]
categories: java io
---

> 作者: Jakob Jenkov 译者: 李璟(jlee381344197@gmail.com)

本小节会简要概括Java IO中的PushbackInputStream，SequenceInputStream和PrintStream。其中，最常用的是PrintStream，System.out和System.err都是PrintStream类型的变量，请查看[Java IO System.in, System.out, System.err](/2015/01/07/java-io-07-system-in-out-error/)浏览更多关于System.out和System.err的信息。

# PushbackInputStream
[原文链接](http://tutorials.jenkov.com/java-io/pushbackinputstream.html)

PushbackInputStream用于解析InputStream内的数据。有时候你需要提前知道接下来将要读取到的字节内容，才能判断用何种方式进行数据解析。PushBackInputStream允许你这么做，你可以把读取到的字节重新推回到InputStream中，以便再次通过read()读取。代码如下：

``` java
PushbackInputStream input = new PushbackInputStream(new FileInputStream("c:\\data\\input.txt"));
int data = input.read();
input.unread(data);
```

可以通过PushBackInputStream的构造函数设置推回缓冲区的大小，代码如下：

``` java
PushbackInputStream input = new PushbackInputStream(new FileInputStream("c:\\data\\input.txt"), 8);
```

这个例子设置了8个字节的缓冲区，意味着你最多可以重新读取8个字节的数据。

# SequenceInputStream
[原文链接](http://tutorials.jenkov.com/java-io/sequenceinputstream.html)

SequenceInputStream把一个或者多个InputStream整合起来，形成一个逻辑连贯的输入流。当读取SequenceInputStream时，会先从第一个输入流中读取，完成之后再从第二个输入流读取，以此推类。代码如下：

``` java
InputStream input1 = new FileInputStream("c:\\data\\file1.txt");
InputStream input2 = new FileInputStream("c:\\data\\file2.txt");
InputStream combined = new SequenceInputStream(input1, input2);
```

通过SequenceInputStream，例子中的2个InputStream使用起来就如同只有一个InputStream一样(译者注：SequenceInputStream的read()方法会在读取到当前流末尾时，关闭流，并把当前流指向逻辑链中的下一个流，最后返回新的当前流的read()值)。

# PrintStream
[原文链接](http://tutorials.jenkov.com/java-io/printstream.html)

PrintStream允许你把格式化数据写入到底层OutputStream中。比如，写入格式化成文本的int，long以及其他原始数据类型到输出流中，而非它们的字节数据。代码如下：

``` java
PrintStream output = new PrintStream(outputStream);
output.print(true);
output.print((int) 123);
output.print((float) 123.456);
output.printf(Locale.UK, "Text + data: %1$", 123);
output.close();
```

PrintStream包含2个强大的函数，分别是format()和printf()(这两个函数几乎做了一样的事情，但是C程序员会更熟悉printf())。

译者注：其中一个printf()函数实现如下：

``` java
public PrintStream printf(String format, Object ... args) {
    return format(format, args);
}
```
> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: 其他字节流(上)](http://ifeve.com/java-io-other-1/)