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

## 磁盘 I/O 工作机制 {#2磁盘IO工作机制outline}

---

前面介绍了基本的 Java I/O 的操作接口，这些接口主要定义了如何操作数据，以及介绍了操作两种数据结构：字节和字符的方式。还有一个关键问题就是数据写到何处，其中一个主要方式就是将数据持久化到物理磁盘，下面将介绍如何将数据持久化到物理磁盘的过程。

我们知道数据在磁盘的唯一最小描述就是文件，也就是说上层应用程序只能通过文件来操作磁盘上的数据，文件也是操作系统和磁盘驱动器交互的一个最小单元。值得注意的是Java 中通常的 File 并不代表一个真实存在的文件对象，当你通过指定一个路径描述符时，它就会返回一个代表这个路径相关联的一个虚拟对象，这个可能是一个真实存在的文件或者是一个包含多个文件的目录。为何要这样设计？因为大部分情况下，我们并不关心这个文件是否真的存在，而是关心这个文件到底如何操作。例如我们手机里通常存了几百个朋友的电话号码，但是我们通常关心的是我有没有这个朋友的电话号码，或者这个电话号码是什么，但是这个电话号码到底能不能打通，我们并不是时时刻刻都去检查，而只有在真正要给他打电话时才会看这个电话能不能用。也就是使用这个电话记录要比打这个电话的次数多很多。

何时真正会要检查一个文件存不存？就是在真正要读取这个文件时，例如 FileInputStream 类是操作一个文件的接口，注意到在创建一个 FileInputStream 对象时，会创建一个 FileDescriptor 对象，其实这个对象就是真正代表一个存在的文件对象的描述，当我们在操作一个文件对象时可以通过 getFD\(\) 方法获取真正操作的与底层操作系统关联的文件描述，例如可以调用 FileDescriptor.sync\(\) 方法将操作系统缓存中的数据强制刷新到物理磁盘中。

下面介绍下如何从磁盘读取一段文本字符，如下图所示：![](/assets/屏幕快照 2018-09-15 下午2.36.56.png)当传入一个文件路径，将会根据这个路径创建一个 File 对象来标识这个文件，然后将会根据这个 File 对象创建真正读取文件的操作对象，这时将会真正创建一个关联真实存在的磁盘文件的文件描述符 FileDescriptor，通过这个对象可以直接控制这个磁盘文件。由于我们需要读取的是字符格式，所以需要 StreamDecoder 类将 byte 解码为 char 格式，至于如何从磁盘驱动器上读取一段数据，由操作系统帮我们完成。

## Java Socket 的工作机制 {#3JavaSocket的工作机制outline}

---

Socket 这个概念没有对应到一个具体的实体，它是描述计算机之间完成相互通信一种抽象功能。打个比方，可以把 Socket 比作为两个城市之间的交通工具，有了它，就可以在城市之间来回穿梭了。交通工具有多种，每种交通工具也有相应的交通规则。Socket 也一样，也有多种。大部分情况下我们使用的都是基于 TCP/IP 的流套接字，它是一种稳定的通信协议。

下图是典型的基于 Socket 的通信的场景：

![](/assets/socket_java.png)  
主机 A 的应用程序要能和主机 B 的应用程序通信，必须通过 Socket 建立连接，而建立 Socket 连接必须需要底层 TCP/IP 协议来建立 TCP 连接。建立 TCP 连接需要底层 IP 协议来寻址网络中的主机。我们知道网络层使用的 IP 协议可以帮助我们根据 IP 地址来找到目标主机，但是一台主机上可能运行着多个应用程序，如何才能与指定的应用程序通信就要通过 TCP 或 UPD 的地址也就是端口号来指定。这样就可以通过一个 Socket 实例唯一代表一个主机上的一个应用程序的通信链路了。

### 建立通信链路

当客户端要与服务端通信，客户端首先要创建一个 Socket 实例，操作系统将为这个 Socket 实例分配一个没有被使用的本地端口号，并创建一个包含本地和远程地址和端口号的套接字数据结构，这个数据结构将一直保存在系统中直到这个连接关闭。在创建 Socket 实例的构造函数正确返回之前，将要进行 TCP 的三次握手协议，TCP 握手协议完成后，Socket 实例对象将创建完成，否则将抛出 IOException 错误。

与之对应的服务端将创建一个 ServerSocket 实例，ServerSocket 创建比较简单只要指定的端口号没有被占用，一般实例创建都会成功，同时操作系统也会为 ServerSocket 实例创建一个底层数据结构，这个数据结构中包含指定监听的端口号和包含监听地址的通配符，通常情况下都是“\*”即监听所有地址。之后当调用 accept\(\) 方法时，将进入阻塞状态，等待客户端的请求。当一个新的请求到来时，将为这个连接创建一个新的套接字数据结构，该套接字数据的信息包含的地址和端口信息正是请求源地址和端口。这个新创建的数据结构将会关联到 ServerSocket 实例的一个未完成的连接数据结构列表中，注意这时服务端与之对应的 Socket 实例并没有完成创建，而要等到与客户端的三次握手完成后，这个服务端的 Socket 实例才会返回，并将这个 Socket 实例对应的数据结构从未完成列表中移到已完成列表中。所以 ServerSocket 所关联的列表中每个数据结构，都代表与一个客户端的建立的 TCP 连接。

### 数据传输 {#N1010F}

传输数据是我们建立连接的主要目的，如何通过 Socket 传输数据，下面将详细介绍。

当连接已经建立成功，服务端和客户端都会拥有一个 Socket 实例，每个 Socket 实例都有一个 InputStream 和 OutputStream，正是通过这两个对象来交换数据。同时我们也知道网络 I/O 都是以字节流传输的。当 Socket 对象创建时，操作系统将会为 InputStream 和 OutputStream 分别分配一定大小的缓冲区，数据的写入和读取都是通过这个缓存区完成的。写入端将数据写到 OutputStream 对应的 SendQ 队列中，当队列填满时，数据将被发送到另一端 InputStream 的 RecvQ 队列中，如果这时 RecvQ 已经满了，那么 OutputStream 的 write 方法将会阻塞直到 RecvQ 队列有足够的空间容纳 SendQ 发送的数据。值得特别注意的是，这个缓存区的大小以及写入端的速度和读取端的速度非常影响这个连接的数据传输效率，由于可能会发生阻塞，所以网络 I/O 与磁盘 I/O 在数据的写入和读取还要有一个协调的过程，如果两边同时传送数据时可能会产生死锁，在后面 NIO 部分将介绍避免这种情况。



内容来源：[深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)

  






