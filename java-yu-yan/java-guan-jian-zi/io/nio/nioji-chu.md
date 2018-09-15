# 理解Selector/Channel/Buffer

## 概念

---

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

## 缓冲区Buffer

---

缓冲区`Buffer`是一个对象，它包含一些要写入或读出的数据。在`NIO`库中，所有数据都是用缓冲区处理的：在读取数据时，从缓冲区读取；在写入数据时，写入到缓冲区中。

`Buffer`实质上是一个数组，最常用的是字节数组（`ByteBuffer`），每一种`Java`基本类型（除了`Boolean`）都对应一种缓冲区：`CharBuffer、ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer`

每个`Buffer`都有以下的属性：

* **capacity**：这个Buffer最多能放多少数据。capacity一般在buffer被创建的时候指定。
* **limit**：在Buffer上进行的读写操作都不能越过这个下标。当写数据到buffer中时，limit一般和capacity相等，当读数据时，limit代表buffer中有效数据的长度。
* **position**：读/写操作的当前下标。当使用buffer的相对位置进行读/写操作时，读/写会从这个下标进行，并在操作完成后，buffer会更新下标的值。
* **mark**：一个临时存放的位置下标。调用mark\(\)会将mark设为当前的position的值，以后调用reset\(\)会将position属性设
  置为mark的值。mark的值总是小于等于position的值，如果将position的值设的比mark小，当前的mark值会被抛弃掉。

这些属性总是满足以下条件：

```
0 <= mark <= position <= limit <= capacity
```

设置这些参数的常用方法：

`Buffer clear()`：把position设为0，把limit设为capacity，**一般在把数据写入Buffer前调用**。

```
public final Buffer clear() {
position = 0; //设置为0
limit = capacity; //极限和容量相同
mark = -1; //取消标记
return this;
}
```

`Buffer flip()`：把limit设为当前position，把position设为0，**一般在从Buffer读出数据前调用**。

```
public final Buffer flip() {
limit = position;
position = 0;
mark = -1;
return this;
}
```

`Buffer rewind()`：把position设为0，limit不变，**一般在把数据重写入Buffer前调用**。

```
public final Buffer rewind() {
position = 0;
mark = -1;
return this;
}
```

另外，`Buffer`对象有可能是只读的，这时，任何对该对象的写操作都会触发一个`ReadOnlyBufferException`。可以使用`isReadOnly()`方法来判断一个`Buffer`是否只读。

## 通道Channel

---

`Channel`是一个通道，网络数据通过`Channel`读取和写入。

`Channel`是双向的，可以用于读、写或读写同时进行，即`Channel`是全双工的。（流则是单向的，只能读或写）

`Channel`可以比流更好地映射操作系统的`API`，特别是在`UNIX`网络编程中，底层操作系统的通道都是全双工的，同时支持读写操作。

`Channel`可以分为两大类：

* `SelectableChannel`：用于网络读写，如`ServerSocketChannel`和`SocketChannel`
* `FileChannel`：用于文件读写

## 多路复用器Selector

---

多路复用器`Selector`是`Java NIO`编程的基础，熟练地掌握`Selector`对于`NIO`编程至关重要

**多路复用器的功能：选择已经就绪的任务**。即`Selector`会不断地轮询注册在其上的`Channel`，如果某个Channel上面发生读或者写事件，这个`Channel`就处于就绪状态，会被`Selector`轮询出来，然后通过SelectionKey就可以获取就绪的`Channel`集合，进行后续的`IO`操作。

一个`Selector`可以同时轮询多个`Channel`。由于JDK使用了epoll\(\)代替传统的select，所以Selector没有最大连接句柄的限制，这意味着只需要一个线程负责Selector的轮询，就可以接入成千上万的客户端，这是一个巨大的进步。

Selector监听Channel中的事件类型如下：

* **connect**：客户端连接服务端事件，对应值为SelectionKey.OPCONNECT\(8\)

* **accept**：服务端接收客户端连接事件，对应值为SelectionKey.OPACCEPT\(16\)

* **read**：读事件，对应值为SelectionKey.OPREAD\(1\)

* **write**：写事件，对应值为SelectionKey.OPWRITE\(4\)

## NIO编程的优势

---

* 客户端发起的连接操作时异步的，可以在Selector中注册连接OP\_CONNECT等待后续结果，
  而不必像BIO一样被同步阻塞。
* SocketChannel的读写操作都是异步的，如果没有可读写的数据，它不会同步等待，直接返回，
  这样IO通信线程就可以处理其他的链路，不需要同步等待这个链路可用。
* 线程模型的优化：同一个Selector可以同时处理成千上万个客户端连接，而且性能不会随着客户端的增加而
  线性下降，因此非常适合做高性能、高负载的网络服务器。

但缺点就是：编码复杂，容易出错。

## 内容来源

[深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)  
《Netty权威指南》  
[深入浅出NIO之Selector实现原理](https://juejin.im/entry/5a422b75f265da430e4f6b99)

