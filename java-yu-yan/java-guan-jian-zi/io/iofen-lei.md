## 概述

---

Java 的 I/O 操作类在包 java.io 下，大概有将近 80 个类，但是这些类可以从不同的角度进行划分。

#### 按照流的流向划分

* 输入流：以InputStream结尾的以及其子类

* 输出流：以OutputStream结尾的的以及其子类

**输入还是输出：**谈这个概念是要有对象的，也就是说是谁的输入流、谁的输出流。对于一个java程序，往外发出信息，就要用输出流。而要从别的地方读取数据，就要用输入流。对于一个文件，别人发来信息我要接受，就要用输入流来读取，而要让A来读取它的内容，那么A就要用输入来读取。总之，两点：一个是具体对象，一个是数据的流向。理解这两个概念就可以明白到底是用输入流还是输出流。这里的输入输出，都是针对java程序而言的，**程序从文件中读取数据就用输入流，程序往文件中写数据就要用输出流**。

#### 按照操作单元划分

* 字节流：xxxStream

* 字符流：Reader/Writer

除了这种划分，[深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html) 中提供了另外一种粒度的划分方式：

* 基于字节操作的 I/O 接口：InputStream 和 OutputStream
* 基于字符操作的 I/O 接口：Writer 和 Reader
* 基于磁盘操作的 I/O 接口：File
* 基于网络操作的 I/O 接口：Socket

#### 按照流的角色划分

* 节点流：直接从指定的位置（如磁盘文件或内存区域）读或写
* * 文 件：FileInputStream、FileOutputStrean、FileReade、FileWriter 文件进行处理的节点流
  * 字符串：StringReader、StringWriter 对字符串进行处理的节点流
  * 数组 ：ByteArrayInputStream、ByteArrayOutputStreamCharArrayReader、CharArrayWriter 对数组进行处理的节点流\(对应的不再是文件，而是内存中的一个数组\)。
  * 管 道 ：PipedInputStream、PipedOutputStream、PipedReaderPipedWriter对管道进行处理的节点流。
  * 父 类： InputStream 、OutputStream、Reader、Writer
* 处理流（又叫过滤流）：以其它输入流/输出流作为它的输入源/输出位置，经过过滤或处理后再以新的输入流/输出流的形式提供给用户
* * 缓冲流：BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter等，增加缓冲功能，避免频繁读写硬盘。
  * 转换流：InputStreamReader、OutputStreamReader等，实现字节流和字符流之间的转换。
  * 数据流 DataInputStream、DataOutputStream 等，提供将基础数据类型写入到文件中或者读取文件。

在计算机的世界中，无论是内存/磁盘/网络传输，最小的存储单元都是字节（一个字节对应8个位），但在程序中，有意义的数据都是以字符形式出现。在Java中的I/O体系中，InputStream/OutputStream对应字节，Reader/Writer对应字符。在此基础之上，针对常见的文件操作，Java提供了File/RandomAccessFile等类；针对常见的网络操作，Java提供了NIO/NIO2等框架。

下面将从输入/输出这个角度，讲解Java中的I/O体系中的InputStream/OutputStream以及Reader/Writer。

## 输入流

---

InputStream 是所有的输入字节流的父类，Reader 是所有的输入字符流的父类，这两个基类的功能基本一样，都是从某处读取或获取流，但是读取的数据单元不同。

```java
public abstract class InputStream implements Closeable {

    //读取一个字节并以整数的形式返回(0~255),如果返回-1已到输入流的末尾。 
    int read();

    //读取一系列字节并存储到一个数组buffer，返回实际读取的字节数，如果读取前已到输入流的末尾返回-1。 
    int read(byte[] buffer);

    //读取length个字节并存储到一个字节数组buffer，从off位置开始存,最多len，返回实际读取的字节数，如果读取前以到输入流的末尾返回-1。 
    int read(byte[] buffer, int off, int len);
}

public abstract class Reader implements Readable, Closeable {

    //读取一个字符并以整数的形式返回(0~255),如果返回-1已到输入流的末尾。 
    int read(); 

    //读取一系列字符并存储到一个数组buffer，返回实际读取的字符数，如果读取前已到输入流的末尾返回-1。 
    int read(char[] cbuf);

    //读取length个字符,并存储到一个数组buffer，从off位置开始存,最多读取len，返回实际读取的字符数，如果读取前以到输入流的末尾返回-1。 
    int read(char[] cbuf, int off, int len);
}
```

**InputStream实现类**

* FileInputStream：从文件获取输入流，实现对文件的读取
* FilterInputStream：过滤流，一般是对原来的输入流进行功能增强
* * BufferedInputStream：提供缓冲功能
  * DataInputStream： 包装为Java中的基本数据类型的流
  * PushbackInputStream：
* PipedInputStream：用于进程间通信
* SequenceInputStream：把多个输入聚合成一个输入流
* StringBufferInputStream：将字符串转换为输入流
* ByteArrayInputStream：将字节数组转换为输入流

**Reader实现类**

* InputStreamReader：将字节流转换为字符流
* * FileReader：与FileInputStream对应，用来读取文件
* BufferedReader：对原输入流提供缓冲

  * LineNumberReader：对原输入流提供按行读取的功能

* CharArrayReader：将字符数组转换为输入流
* StringReader：将字符串转换为输入流
* FilterReader：对原字符流进行过滤
  * PushbackReader：
* PipedReader：

## 输出流

---

OutputStream是所有的输出字节流的父类，Writer 是所有的输出字符流的父类，这两个基类的功能基本一样，都是从将流写入到某处，但是操作的数据单元不同。

```java
public abstract class OutputStream implements Closeable, Flushable {
    //向输出流中写入一个字节数据,该字节数据为参数b的低8位。 
    void write(int b) ; 

    //将一个字节类型的数组中的数据写入输出流。 
    void write(byte[] b); 

    //将一个字节类型的数组中的从指定位置（off）开始的,len个字节写入到输出流。 
    void write(byte[] b, int off, int len); 

    //将输出流中缓冲的数据全部写出到目的地。 
    void flush();
}

public abstract class Writer implements Appendable, Closeable, Flushable {
    //向输出流中写入一个字符数据,该字节数据为参数b的低16位。 
    void write(int c); 

    //将一个字符类型的数组中的数据写入输出流， 
    void write(char[] cbuf) 

    //将一个字符类型的数组中的从指定位置（offset）开始的,length个字符写入到输出流。 
    void write(char[] cbuf, int offset, int length);

    //将一个字符串中的字符写入到输出流。 
    void write(String string); 

    //将一个字符串从offset开始的length个字符写入到输出流。 
    void write(String string, int offset, int length); 
    //将输出流中缓冲的数据全部写出到目的地。 

    void flush()
}
```

**OutputStream实现类**

* FileOutputStream：将数据写入文件中
* FilterOutputStream：对输出流进行过滤
* * BufferedOutputStream：提供缓冲功能
  * DataOutputStream：将数据写入到Java基本类型中
  * PrintStream：提供打印功能，即将数据写入到控制台
* PipedOutputStream：todo
* ObjectOutputStream：将数据写入到一个Java对象中
* ByteArrayOutputStream：将数据写入到内存中的字节数组中

**Writer实现类**

* OutputStreamWriter：从字符到字节的转换
* * FileWriter：将数据写入到文件中
* CharArrayWriter：将数据写入到内存中的字符数组中

* StringWriter：将数组写入到一个字符串中
* PipedWriter：todo
* BufferedWriter：对输出流提供缓冲功能
* PrintWriter：将数据写入到控制台，即提供打印功能
* FilterWriter：对输出流进行过滤

## 总结

---

对于Java的I/O诸多实现类的命名，如果将实现类的名字分为前后两部分，则可以从名字中获取这样的信息：

* 前半部分
* * 流的数据源/目的地
  * * 以File开头的，说明操作的是文件
    * 以String开头的，说明操作的是字符串
    * 以Char开头的，说明操作的是字符
    * 以Byte开头的，说明操作的是字节
    * 以Data开头的，说明操作的Java中的基本数据类型的数据
    * 以Object开头的，说明操作的是Java中的对象
  * 流的功能
  * * 以Buffer开头的，说明该类是对另外一个流提供缓冲功能（输入流/输出流）
    * 以Print开头的，说明该类将数据写入到控制台，即提供打印功能（输出流）
* 后半部分
* * zijis输入/输出
  * 输入/输出
  * * 以Input/Reader结束的，说明是输入流
  * * 以Output/Writer结束的，说明是输出流

**Java I/O类库的设计是装饰模式应用的经典**。即不同的输入/输出流，可以被具有不同功能的类进行装饰，实现功能增强，如下：

```java
OutputStream out = new BufferedOutputStream(new ObjectOutputStream(new FileOutputStream("fileName"))；
```

# 参考

[深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)

[Java IO流详解（二）——IO流的框架体系](http://lruheng.com/2017/02/22/Java-IO流详解（二）——IO流的框架体系/)

[Java IO详解](https://segmentfault.com/a/1190000007120335)

[Java IO教程](http://ifeve.com/java-io/)：是名副其实的教程，对每个类介绍的都很详细

[Java输入输出流](https://blog.csdn.net/hguisu/article/details/7418161)

