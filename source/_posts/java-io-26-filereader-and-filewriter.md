---
title: 26 - Java IO FileReader和FileWriter
date: 2015-01-26 17:36:55
tags: [java, java io]
categories: java io
---

> 作者: Jakob Jenkov 译者: 李璟(jlee381344197@gmail.com)

本章节将简要介绍FileReader和FileWriter。与FileInputStream和FileOutputStream类似，FileReader与FileWriter用于处理文件内容。

# FileReader
[原文链接](http://tutorials.jenkov.com/java-io/filereader.html)

FileReader能够以字符流的形式读取文件内容。除了读取的单位不同之外(译者注：FileReader读取字符，FileInputStream读取字节)，FileReader与FileInputStream并无太大差异，也就是说，FileReader用于读取文本。根据不同的编码方案，一个字符可能会相当于一个或者多个字节。代码如下：

``` java
Reader reader = new FileReader("c:\\data\\input-text.txt");
int data = reader.read();
while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = reader.read();
}
reader.close();
```

注意：为了清晰，代码忽略了一些必要的异常处理。想了解更多异常处理的信息，请参考[Java IO异常处理](/2015/01/12/java-io-12-exception/)。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注：0~65535)。如果方法返回-1，表明FileReader中已经没有剩余可读取字符，此时可以关闭FileReader。-1是一个int类型，不是byte或者char类型，这是不一样的。

FileReader拥有其他可选的构造函数，能够让你使用不同的方式读取文件，更多内容请查看官方文档。

FileReader会假设你想使用你所使用的JVM的版本的默认编码处理字节流，但是这通常不是你想要的，你可以手动设置编码方案。

如果你想明确指定一种编码方案，利用InputStreamReader配合FileInputStream来替代FileReader(译者注：FileReader没有可以指定编码的构造函数)。InputStreamReader可以让你设置编码处理从底层文件中读取的字节。

# FileWriter
[原文链接](http://tutorials.jenkov.com/java-io/filewriter.html)

FileWriter能够把数据以字符流的形式写入文件。同样是处理文件，FileWriter处理字符，FileOutputStream处理字节。根据不同的编码方案，一个字符可能会相当于一个或者多个字节。代码如下：

``` java
Writer writer = new FileWriter("c:\\data\\output.txt");
while(moreData) {
    String data = getMoreData();
    write.write(data);
}
writer.close();
```

处理文件都会碰到的一个问题是，当前写入的数据是覆盖原文件内容还是追加到文件末尾。当你创建一个FileWriter之后，你可以通过使用不同构造函数实现你的不同目的。

以下的构造函数取文件名作为参数，将会新写入的内容将会覆盖该文件：

``` java
Writer writer = new FileWriter("c:\\data\\output.txt");
```

以下的构造函数取文件名和一个布尔变量作为参数，布尔值表明你是想追加还是覆盖该文件。例子如下：

``` java
Writer writer = new FileWriter("c:\\data\\output.txt", true); //appends to file
Writer writer = new FileWriter("c:\\data\\output.txt", false); //overwrites file
```

同样，FileWriter不能指定编码，可以通过OutputStreamWriter配合FileOutputStream替代FileWriter。

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: FileReader和FileWriter](http://ifeve.com/java-io-filereader%E5%92%8Cfilewriter/)