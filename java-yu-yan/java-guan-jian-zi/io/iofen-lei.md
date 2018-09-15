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

基于字节的 I/O 操作接口输入和输出分别是：InputStream 和 OutputStream。

**InputStream**

InputStream 是所有的输入字节流的父类，它是一个抽象类，主要包含三个方法：

```java
//读取一个字节并以整数的形式返回(0~255),如果返回-1已到输入流的末尾。 
int read()；

//读取一系列字节并存储到一个数组buffer，返回实际读取的字节数，如果读取前已到输入流的末尾返回-1。 
int read(byte[] buffer)；

//读取length个字节并存储到一个字节数组buffer，从off位置开始存,最多len，返回实际读取的字节数，如果读取前以到输入流的末尾返回-1。 
int read(byte[] buffer, int off, int len) ；
```

在执行完流操作后，要调用`close()`方法来显式地关闭输入流，因为程序里打开的IO资源不属于内存资源，垃圾回收机制无法回收该资源，所以应该显式关闭文件IO资源。

21212

* FileInputStream：
* FilterInputStream：
* PipedInputStream：
* SequenceInputStream：将多个输入流合并成一个输入流，通过该类包装后形成新的一个总的输入流。在拷贝多个文件到一个目标文件的时候是非常有用的。可用使用很少的代码实现
* StringBufferInputStream
* ByteArrayInputStream

**OutputStream**

* FileOutputStream
* FilterOutputStream
* PipedOutputStream
* ObjectOutputStream
* ByteArrayOutputStream

## 基于字符的 I/O 操作接口

---

不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符，所以 I/O 操作的都是字节而不是字符，但是为啥有操作字符的 I/O 接口呢？这是因为我们的程序中通常操作的数据都是以字符形式，为了操作方便当然要提供一个直接写字符的 I/O 接口，如此而已。我们知道字符到字节必须要经过编码转换，而这个编码又非常耗时，而且还会经常出现乱码问题，所以 I/O 的编码问题经常是让人头疼的问题。

**Reader**

Reader 是所有的输入字符流的父类，它是一个抽象类，主要包含三个方法（与InputStream所提供的方法类似，只不过读取的数据单元变为了字符）：

```java
//读取一个字符并以整数的形式返回(0~255),如果返回-1已到输入流的末尾。 
int read()； 

//读取一系列字符并存储到一个数组buffer，返回实际读取的字符数，如果读取前已到输入流的末尾返回-1。 
int read(char[] cbuf)； 

//读取length个字符,并存储到一个数组buffer，从off位置开始存,最多读取len，返回实际读取的字符数，如果读取前以到输入流的末尾返回-1。 
int read(char[] cbuf, int off, int len)
```

* InputStreamReader
  * FileReader
* BufferedReader
  * LineNumberReader
* CharArrayReader
* StringReader
* FilterReader
  * PushBackReader
* PipedReader

**Writer**

Writer 类提供了一个抽象方法 write\(char cbuf\[\], int off, int len\) 由子类去实现。

* OutputStreamWriter
  * FileWriter）
* CharArrayWriter
* StringWriter
* PipedWriter
* BufferedWriter
* PrintWriter
* FilterWriter

不管是 Writer 还是 Reader 类，它们都只定义了读取或写入的数据字符的方式，也就是怎么写或读，但是并没有规定数据要写到哪去，或者从哪里读取。

## 字节与字符的转化接口

---

磁盘存取或网络传输都是以字节进行的，所以必须要有字节到字符或者字符到字节的转化。

**字节到字符**

InputStreamReader是字节到字符的转化桥梁，InputStream 到 Reader 的过程要指定编码字符集，否则将采用操作系统默认字符集，很可能会出现乱码问题。StreamDecoder 正是完成字节到字符的解码的实现类。

也就是当你用如下方式读取一个文件时，FileReader 类就是按照上面的工作方式读取文件的，FileReader 是继承了 InputStreamReader类，实际上是读取文件流，然后通过 StreamDecoder 解码成 char，只不过这里的解码字符集是默认字符集。

```java
try { 
    StringBuffer str = new StringBuffer(); 
    char[] buf = new char[1024]; 
    FileReader f = new FileReader("file"); 
    while(f.read(buf)>0){ 
        str.append(buf); 
    } 
    str.toString(); 
} catch (IOException e) {
}
```

**字符到字节**

Writer的写入过程与之类似：通过 OutputStreamWriter 类完成，字符到字节的编码过程，由 StreamEncoder 完成编码过程。

**Java I/O类库的设计是装饰模式应用的经典**。即不同的输入/输出流，可以被具有不同功能的类进行装饰，实现功能增强，如下：

```java
OutputStream out = new BufferedOutputStream(new ObjectOutputStream(new FileOutputStream("fileName"))；
```

# 参考

[深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)

[Java IO流详解（二）——IO流的框架体系](http://lruheng.com/2017/02/22/Java-IO流详解（二）——IO流的框架体系/)

