# I/O

---

I/O 问题是任何编程语言都无法回避的问题，因为 I/O 是机器获取和交换信息的主要渠道。

Java 的 I/O 操作类在包 java.io 下，大概有将近 80 个类，但是这些类大概可以分成四组，分别是：

1. 基于字节操作的 I/O 接口：InputStream 和 OutputStream
2. 基于字符操作的 I/O 接口：Writer 和 Reader
3. 基于磁盘操作的 I/O 接口：File
4. 基于网络操作的 I/O 接口：Socket

前两组主要是根据传输数据的数据格式，后两组主要是根据传输数据的方式。虽然 Socket 类并不在 java.io 包下，但是我仍然把它们划分在一起，因为我个人认为 I/O 的核心问题要么是数据格式影响 I/O 操作，要么是传输方式影响 I/O 操作，也就是将什么样的数据写到什么地方的问题，I/O 只是人与机器或者机器与机器交互的手段，除了在它们能够完成这个交互功能外，我们关注的就是如何提高它的运行效率了，而数据格式和传输方式是影响效率最关键的因素了。

**概念：输入还是输出？**

谈这个概念是要有对象的，也就是说是谁的输入流、谁的输出流。对于一个java程序，往外发出信息，就要用输出流。而要从别的地方读取数据，就要用输入流。对于一个文件，别人发来信息我要接受，就要用输入流来读取，而要让A来读取它的内容，那么A就要用输入来读取。总之，两点：一个是具体对象，一个是数据的流向。理解这两个概念就可以明白到底是用输入流还是输出流。这里的输入输出，都是针对java程序而言的，程序从文件中读取数据就用输入流，程序往文件中写数据就要用输出流。

## 基于字节的 I/O 操作接口

---

基于字节的 I/O 操作接口输入和输出分别是：InputStream 和 OutputStream，InputStream 输入流的实现类如下所示：

| 基于字节的I/O输入流 | 说明 |
| :--- | :--- |
| FileInputStream |  |
| FilterInputStream |  |
| PipedInputStream |  |
| SequenceInputStream |  |
| StringBufferInputStream |  |
| ByteArrayInputStream |  |

输入流根据数据类型和操作方式又被划分成若干个子类，每个子类分别处理不同操作类型。

OutputStream 输出流的如下所示：

| 基于字节的I/O输出流 | 说明 |
| :--- | :--- |
| FileOutputStream |  |
| FilterOutputStream |  |
| PipedOutputStream |  |
| ObjectOutputStream |  |
| ByteArrayOutputStream |  |

**Java I/O类库的设计是装饰模式应用的经典**。即不同的输入/输出流，可以被具有不同功能的类进行装饰，实现功能增强，如下：

```java
OutputStream out = new BufferedOutputStream(new ObjectOutputStream(new FileOutputStream("fileName"))；
```

还有一点是流最终写到什么地方必须要指定，要么是写到磁盘要么是写到网络中，其实从上面的类图中我们发现，写网络实际上也是写文件，只不过写网络还有一步需要处理就是底层操作系统再将数据传送到其它地方而不是本地磁盘。

## 基于字符的 I/O 操作接口

---

不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符，所以 I/O 操作的都是字节而不是字符，但是为啥有操作字符的 I/O 接口呢？这是因为我们的程序中通常操作的数据都是以字符形式，为了操作方便当然要提供一个直接写字符的 I/O 接口，如此而已。我们知道字符到字节必须要经过编码转换，而这个编码又非常耗时，而且还会经常出现乱码问题，所以 I/O 的编码问题经常是让人头疼的问题。关于 I/O 编码问题请参考另一篇文章[《深入分析](http://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/)[Java](http://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/)[中的中文编码问题》](http://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/)。

**Reader**

Reader提供了一个抽象方法 int read\(char cbuf\[\], int off, int len\)，返回读到的 n 个字节数

| 基于字符的I/O输入流 | 说明 |
| :--- | :--- |
| InputStreamReader |  |
| FileReader\(InputStreamReader的子类\) |  |
| BufferedReader |  |
| LineNumberReader\(BufferedReader的子类\) |  |
| CharArrayReader |  |
| StringReader |  |
| FilterReader |  |
| PushBackReader\(FilterReader的子类\) |  |
| PipedReader |  |

**Writer**

Writer 类提供了一个抽象方法 write\(char cbuf\[\], int off, int len\) 由子类去实现。

| 基于字符的I/O输出流 | 说明 |
| :--- | :--- |
| OutputStreamWriter（子类FileWriter） |  |
| CharArrayWriter |  |
| StringWriter |  |
| PipedWriter |  |
| BufferedWriter |  |
| PrintWriter |  |
| FilterWriter |  |

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





内容来源：[深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)

  






