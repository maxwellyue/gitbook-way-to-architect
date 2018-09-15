## NIO {#4NIO的工作方式outline}

---

### BIO 的问题 {#N1011D}

BIO 即阻塞 I/O，不管是磁盘 I/O 还是网络 I/O，数据在写入 OutputStream 或者从 InputStream 读取时都有可能会阻塞。一旦有线程阻塞将会失去 CPU 的使用权，这在当前的大规模访问量和有性能要求情况下是不能接受的。虽然当前的网络 I/O 有一些解决办法，如一个客户端一个处理线程，出现阻塞时只是一个线程阻塞而不会影响其它线程工作，还有为了减少系统线程的开销，采用线程池的办法来减少线程创建和回收的成本，但是有一些使用场景仍然是无法解决的。如当前一些需要大量 HTTP 长连接的情况，像淘宝现在使用的 Web 旺旺项目，服务端需要同时保持几百万的 HTTP 连接，但是并不是每时每刻这些连接都在传输数据，这种情况下不可能同时创建这么多线程来保持连接。即使线程的数量不是问题，仍然有一些问题还是无法避免的。如这种情况，我们想给某些客户端更高的服务优先级，很难通过设计线程的优先级来完成，另外一种情况是，我们需要让每个客户端的请求在服务端可能需要访问一些竞争资源，由于这些客户端是在不同线程中，因此需要同步，而往往要实现这些同步操作要远远比用单线程复杂很多。以上这些情况都说明，我们需要另外一种新的 I/O 操作方式。

### NIO 的工作机制 {#N10124}

我们先看一下 NIO 涉及到的关联类图，如下：

![](/assets/nio_class.png)

上图中有两个关键类：Channel 和 Selector，它们是 NIO 中两个核心概念。我们还用前面的城市交通工具来继续比喻 NIO 的工作方式，这里的 Channel 要比 Socket 更加具体，它可以比作为某种具体的交通工具，如汽车或是高铁等，而 Selector 可以比作为一个车站的车辆运行调度系统，它将负责监控每辆车的当前运行状态：是已经出战还是在路上等等，也就是它可以轮询每个 Channel 的状态。这里还有一个 Buffer 类，它也比 Stream 更加具体化，我们可以将它比作为车上的座位，Channel 是汽车的话就是汽车上的座位，高铁上就是高铁上的座位，它始终是一个具体的概念，与 Stream 不同。Stream 只能代表是一个座位，至于是什么座位由你自己去想象，也就是你在去上车之前并不知道，这个车上是否还有没有座位了，也不知道上的是什么车，因为你并不能选择，这些信息都已经被封装在了运输工具（Socket）里面了，对你是透明的。NIO 引入了 Channel、Buffer 和 Selector 就是想把这些信息具体化，让程序员有机会控制它们，如：当我们调用 write\(\) 往 SendQ 写数据时，当一次写的数据超过 SendQ 长度是需要按照 SendQ 的长度进行分割，这个过程中需要有将用户空间数据和内核地址空间进行切换，而这个切换不是你可以控制的。而在 Buffer 中我们可以控制 Buffer 的 capacity，并且是否扩容以及如何扩容都可以控制。

理解了这些概念后我们看一下，实际上它们是如何工作的，下面是典型的一段 NIO 代码：

```java
public void selector() throws IOException {
    ByteBuffer buffer = ByteBuffer.allocate(1024);//创建缓冲器
    Selector selector = Selector.open();//调用 Selector 的静态工厂创建一个选择器
    ServerSocketChannel ssc = ServerSocketChannel.open();//创建一个服务端的Channel
    ssc.configureBlocking(false);//设置为非阻塞方式
    ssc.socket().bind(new InetSocketAddress(8080));//将channel绑定到一个Socket对象
    ssc.register(selector, SelectionKey.OP_ACCEPT);//将channel注册到选择器上，并设置监听的事件类型
    while (true) {
        Set selectedKeys = selector.selectedKeys();//取得所有key集合
        Iterator it = selectedKeys.iterator();
        while (it.hasNext()) {
            SelectionKey key = (SelectionKey) it.next();
            if ((key.readyOps() & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT) {
                ServerSocketChannel ssChannel = (ServerSocketChannel) key.channel();
                SocketChannel sc = ssChannel.accept();//接受到服务端的请求
                sc.configureBlocking(false);
                sc.register(selector, SelectionKey.OP_READ);
                it.remove();
            } else if 
            ((key.readyOps() & SelectionKey.OP_READ) == SelectionKey.OP_READ) {
                SocketChannel sc = (SocketChannel) key.channel();
                while (true) {
                    buffer.clear();
                    int n = sc.read(buffer);//读取数据
                    if (n <= 0) {
                        break;
                    }
                    buffer.flip();
                }
                it.remove();
            }
        }
    }
}
```

调用 Selector 的静态工厂创建一个选择器，创建一个服务端的 Channel 绑定到一个 Socket 对象，并把这个通信信道注册到选择器上，把这个通信信道设置为非阻塞模式。然后就可以调用 Selector 的 selectedKeys 方法来检查已经注册在这个选择器上的所有通信信道是否有需要的事件发生，如果有某个事件发生时，将会返回所有的 SelectionKey，通过这个对象 Channel 方法就可以取得这个通信信道对象从而可以读取通信的数据，而这里读取的数据是 Buffer，这个 Buffer 是我们可以控制的缓冲器。

在上面的这段程序中，是将 Server 端的监听连接请求的事件和处理请求的事件放在一个线程中，但是在实际应用中，我们通常会把它们放在两个线程中，一个线程专门负责监听客户端的连接请求，而且是阻塞方式执行的；另外一个线程专门来处理请求，这个专门处理请求的线程才会真正采用 NIO 的方式，像 Web 服务器 Tomcat 和 Jetty 都是这个处理方式。

### Buffer 的工作方式 {#N10153}

上面介绍了 Selector 将检测到有通信信道 I/O 有数据传输时，通过 selelct\(\) 取得 SocketChannel，将数据读取或写入 Buffer 缓冲区。下面讨论一下 Buffer 如何接受和写出数据？

Buffer 可以简单的理解为一组基本数据类型的元素列表，它通过几个变量来保存这个数据的当前位置状态，如下所示：

| 变量 | **说明** |
| :--- | :--- |
| capacity | 缓冲区数组的总长度 |
| position | 下一个要操作的数据元素的位置 |
| limit | 数组中不可操作的下一个元素的位置，limit&lt;=capacity |
| mark | 用于记录当前 position 的前一个位置或者默认是 0 |

在实际操作数据时它们有如下关系：

##### ![](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/image023.jpg "Figure xxx. Requires a heading") {#N10189}

我们通过 ByteBuffer.allocate\(11\) 方法创建一个 11 个 byte 的数组缓冲区，初始状态如上图所示，position 的位置为 0，capacity 和 limit 默认都是数组长度。当我们写入 5 个字节时位置变化如下图所示：

##### ![](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/image025.jpg "Figure xxx. Requires a heading") {#N10194}

这时我们需要将缓冲区的 5 个字节数据写入 Channel 通信信道，所以我们需要调用 byteBuffer.flip\(\) 方法，数组的状态又发生如下变化：

##### ![](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/image027.jpg "Figure xxx. Requires a heading") {#N1019F}

这时底层操作系统就可以从缓冲区中正确读取这 5 个字节数据发送出去了。在下一次写数据之前我们在调一下 clear\(\) 方法。缓冲区的索引状态又回到初始位置。

这里还要说明一下 mark，当我们调用 mark\(\) 时，它将记录当前 position 的前一个位置，当我们调用 reset 时，position 将恢复 mark 记录下来的值。

还有一点需要说明，通过 Channel 获取的 I/O 数据首先要经过操作系统的 Socket 缓冲区再将数据复制到 Buffer 中，这个的操作系统缓冲区就是底层的 TCP 协议关联的 RecvQ 或者 SendQ 队列，从操作系统缓冲区到用户缓冲区复制数据比较耗性能，Buffer 提供了另外一种直接操作操作系统缓冲区的的方式即 ByteBuffer.allocateDirector\(size\)，这个方法返回的 byteBuffer 就是与底层存储空间关联的缓冲区，它的操作方式与 linux2.4 内核的 sendfile 操作方式类似。

**总结**

**Selector和Channel**

一个Selector管理多个Channel（一个Channel代表一个通信通道或简单的说是连接）：将Channel注册到Selector，并声明需要让Selector监听这个Channel中的事件类型。事件类型如下：

* **connect**：客户端连接服务端事件，对应值为SelectionKey.OPCONNECT\(8\) 

* **accept**：服务端接收客户端连接事件，对应值为SelectionKey.OPACCEPT\(16\) 

* **read**：读事件，对应值为SelectionKey.OPREAD\(1\) 

* **write**：写事件，对应值为SelectionKey.OPWRITE\(4\)

**ByteBuffer**

数据的读取与写入均在这个Buffer中，它有3个重要的位置信息：capacity/limit/position。写入的时候，首先clear\(\)，position=0，limit=capacity，写完数据后，position为写入数据的最后一位的下一位；读取的时候，flip\(\) 一下，position回到0，limit为buffer中数据的最后一位。  










内容来源：[深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)

参考内容：[深入浅出NIO之Selector实现原理](https://juejin.im/entry/5a422b75f265da430e4f6b99)

  


