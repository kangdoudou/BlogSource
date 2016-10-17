---
title: 23 - Java IO 序列化与ObjectInputStream、ObjectOutputStream
date: 2015-01-23 17:36:55
tags: [java, java io]
categories: java io
---

> 作者：Jakob Jenkov  译者： 李璟(jlee381344197@gmail.com)

本小节会简要概括Java IO中的序列化以及涉及到的流，主要包括ObjectInputStream和ObjectOutputStream。

# Serializable
[原文链接](http://tutorials.jenkov.com/java-io/serializable.html)

如果你希望类能够序列化和反序列化，必须实现Serializable接口，就像所展示的ObjectInputStream和ObjectOutputStream例子一样。

对象序列化本身就是一个主题。Java IO系列教程主要关注流、reader和writer，所以我不会深入探讨对象序列化的细节。并且，目前在网上已经有很多文章探讨了对象序列化，我将给出几个深入分析的资料链接，不再赘述。链接：[传送门](http://java.sun.com/developer/technicalArticles/Programming/serialization/)

# ObjectInputStream
[原文链接](http://tutorials.jenkov.com/java-io/objectinputstream.html)

ObjectInputStream能够让你从输入流中读取Java对象，而不需要每次读取一个字节。你可以把InputStream包装到ObjectInputStream中，然后就可以从中读取对象了。代码如下：

``` java
ObjectInputStream input = new ObjectInputStream(new FileInputStream("object.data"));
MyClass object = (MyClass) input.readObject(); //etc.
input.close();
```

在这个例子中，你读取的对象必须是MyClass的一个实例，并且必须事先通过ObjectOutputStream序列化到“object.data”文件中。(译者注：ObjectInputStream和ObjectOutputStream还有许多read和write方法，比如readInt、writeLong等等，详细信息请查看[官方文档](http://docs.oracle.com/javase/7/docs/api/))

在你序列化和反序列化一个对象之前，该对象的类必须实现了java.io.Serializable接口。

# ObjectOutputStream
[原文链接](http://tutorials.jenkov.com/java-io/objectoutputstream.html)

ObjectOutputStream能够让你把对象写入到输出流中，而不需要每次写入一个字节。你可以把OutputStream包装到ObjectOutputStream中，然后就可以把对象写入到该输出流中了。代码如下：

``` java
ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream("object.data"));
MyClass object = new MyClass();  output.writeObject(object); //etc.
output.close();
```

例子中序列化的对象object现在可以从ObjectInputStream中读取了。

同样，在你序列化和反序列化一个对象之前，该对象的类必须实现了java.io.Serializable接口。

原创文章，转载请注明： 转载自并发编程网 – ifeve.com本文链接地址: Java IO: 序列化与ObjectInputStream、ObjectOutputStream

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: 序列化与ObjectInputStream、ObjectOutputStream](http://ifeve.com/java-io-s-objectinputstream-objectoutputstream/)