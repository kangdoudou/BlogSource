---
title: 30 - Java IO 其他字符流(下)
date: 2015-01-30 17:36:55
tags: [java, java io]
categories: java io
---

> 作者: Jakob Jenkov 译者: 李璟(jlee381344197@gmail.com)

本小节会简要概括Java IO中的PushbackReader，LineNumberReader，StreamTokenizer，PrintWriter，StringReader，StringWriter。

# PushbackReader
[原文链接](http://tutorials.jenkov.com/java-io/pushbackreader.html)

PushbackReader与PushbackInputStream类似，唯一不同的是PushbackReader处理字符，PushbackInputStream处理字节。代码如下：

``` java
PushbackReader reader = new PushbackReader(new FileReader("c:\\data\\input.txt"));
int data = reader.read();
reader.unread(data);
```

同样可以设置缓冲区大小，代码如下：

``` java
PushbackReader reader = new PushbackReader(new FileReader("c:\\data\\input.txt"), 8);
```

# LineNumberReader
[原文链接](http://tutorials.jenkov.com/java-io/linenumberreader.html)

LineNumberReader是记录了已读取数据行号的BufferedReader。默认情况下，行号从0开始，当LineNumberReader读取到行终止符时，行号会递增(译者注：换行\n，回车\r，或者换行回车\n\r都是行终止符)。

你可以通过getLineNumber()方法获取当前行号，通过setLineNumber()方法设置当前行数(译者注：setLineNumber()仅仅改变LineNumberReader内的记录行号的变量值，不会改变当前流的读取位置。流的读取依然是顺序进行，意味着你不能通过setLineNumber()实现流的跳跃读取)。代码如下：

``` java
LineNumberReader reader = new LineNumberReader(new FileReader("c:\\data\\input.txt"));
int data = reader.read();
while(data != -1){
    char dataChar = (char) data;
    data = reader.read();
    int lineNumber = reader.getLineNumber();
}
```

如果解析的文本有错误，LineNumberReader可以很方便地定位问题。当你把错误报告给用户时，如果能够同时把出错的行号提供给用户，用户就能迅速发现并且解决问题。

# StreamTokenizer
[原文链接](http://tutorials.jenkov.com/java-io/streamtokenizer.html)

StreamTokenizer(译者注：请注意不是StringTokenizer)可以把输入流(译者注：InputStream和Reader。通过InputStream构造StreamTokenizer的构造函数已经在JDK1.1版本过时，推荐将InputStream转化成Reader，再利用此Reader构造StringTokenizer)分解成一系列符号。比如，句子”Mary had a little lamb”的每个单词都是一个单独的符号。

当你解析文件或者计算机语言时，为了进一步的处理，需要将解析的数据分解成符号。通常这个过程也称作分词。

通过循环调用nextToken()可以遍历底层输入流的所有符号。在每次调用nextToken()之后，StreamTokenizer有一些变量可以帮助我们获取读取到的符号的类型和值。这些变量是：

ttype 读取到的符号的类型(字符，数字，或者行结尾符)

sval 如果读取到的符号是字符串类型，该变量的值就是读取到的字符串的值

nval 如果读取到的符号是数字类型，该变量的值就是读取到的数字的值

代码如下：

``` java
StreamTokenizer tokenizer = new StreamTokenizer(new StringReader("Mary had 1 little lamb..."));
while(tokenizer.nextToken() != StreamTokenizer.TT_EOF){
    if(tokenizer.ttype == StreamTokenizer.TT_WORD) {
        System.out.println(tokenizer.sval);
    } else if(tokenizer.ttype == StreamTokenizer.TT_NUMBER) {
        System.out.println(tokenizer.nval);
    } else if(tokenizer.ttype == StreamTokenizer.TT_EOL) {
        System.out.println();
    }
}
```

译者注：TT_EOF表示流末尾，TT_EOL表示行末尾。

StreamTokenizer可以识别标示符，数字，引用的字符串，和多种注释类型。你也可以指定何种字符解释成空格、注释的开始以及结束等。在StreamTokenizer开始解析之前，所有的功能都可以进行配置。请查阅官方文档获取更多信息。

# PrintWriter
[原文链接](http://tutorials.jenkov.com/java-io/printwriter.html)

与[PrintStream](/2015/01/29/java-io-29-other-1/#PrintStream)类似，PrintWriter可以把格式化后的数据写入到底层writer中。由于内容相似，不再赘述。

值得一提的是，PrintWriter有更多种构造函数供使用者选择，除了可以输出到文件、Writer以外，还可以输出到OutputStream中(译者注：PrintStream只能把数据输出到文件和OutputStream)。

# StringReader
[原文链接](http://tutorials.jenkov.com/java-io/stringreader.html)

StringReader能够将原始字符串转换成Reader，代码如下：

``` java
Reader reader = new StringReader("input string...");
int data = reader.read();
while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = reader.read();
}
reader.close();
```

# StringWriter
[原文链接](http://tutorials.jenkov.com/java-io:-stringwriter.html)

StringWriter能够以字符串的形式从Writer中获取写入到其中数据，代码如下：

``` java
StringWriter writer = new StringWriter();
//write characters to writer.
String data = writer.toString();
StringBuffer dataBuffer = writer.getBuffer();
```
toString()方法能够获取StringWriter中的字符串数据。

getBuffer()方法能够获取StringWriter内部构造字符串时所使用的StringBuffer对象。

> 转载自[并发编程网 – ifeve.com](http://ifeve.com/) 本文链接地址: [Java IO: 其他字符流(下)](http://ifeve.com/java-io-other/)