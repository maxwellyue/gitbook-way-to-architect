### 传统的BIO编程

网络编程的基本模型都是`Client/Server`模型：即两个进程之间的通信，
其中，
服务端提供位置信息（`IP`和`Port`），客户端通过连接操作向服务端监听的地址发起连接请求，通过三次握手建立连接。

在基于BIO的开发中，
①`ServerSocket`负责绑定`IP`地址，并启动监听端口；
②`Socket`负责发起连接操作；
③连接成功后，双方通过输入和输出流来进行同步阻塞通信

---
### 基于NIO的非阻塞编程

##### 缓冲区Buffer
缓冲区`Buffer`是一个对象，它包含一些要写入或读出的数据。
在`NIO`库中，所有数据都是用缓冲区处理的：在读取数据时，从缓冲区读取；在写入数据时，写入到缓冲区中。
`Buffer`实质上是一个数组，最常用的是字节数组（`ByteBuffer`），每一种`Java`基本类型（除了`Boolean`）都对应一种缓冲区：`CharBuffer、ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer`

每个`Buffer`都有以下的属性：
* **capacity**：这个Buffer最多能放多少数据。capacity一般在buffer被创建的时候指定。
* **limit**：在Buffer上进行的读写操作都不能越过这个下标。当写数据到buffer中时，limit一般和capacity相等，当读数据时，limit代表buffer中有效数据的长度。
* **position**：读/写操作的当前下标。当使用buffer的相对位置进行读/写操作时，读/写会从这个下标进行，并在操作完成后，buffer会更新下标的值。
* **mark**：一个临时存放的位置下标。调用mark()会将mark设为当前的position的值，以后调用reset()会将position属性设
置为mark的值。mark的值总是小于等于position的值，如果将position的值设的比mark小，当前的mark值会被抛弃掉。

这些属性总是满足以下条件：
```

0 <= mark <= position <= limit <= capacity

```
设置这些参数的常用方法：

`Buffer clear()`：把position设为0，把limit设为capacity，**一般在把数据写入Buffer前调用**。

```
public final Buffer clear() {
    position = 0;      //设置为0
    limit = capacity;    //极限和容量相同
    mark = -1;   //取消标记
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


另外，`Buffer`对象有可能是只读的，这时，任何对该对象的写操作都会触发一个`ReadOnlyBufferException`。
`isReadOnly()`方法可以用来判断一个`Buffer`是否只读。



##### 通道Channel
`Channel`是一个通道，网络数据通过`Channel`读取和写入。
* `Channel`是双向的，可以用于读、写或读写同时进行，即`Channel`是全双工的。（流则是单向的，只能读或写）
* `Channel`可以比流更好地映射操作系统的`API`，特别是在`UNIX`网络编程中，底层操作系统的通道都是全双工的，同时支持读写操作。
* `Channel`可以分为两大类
  * `SelectableChannel`：用于网络读写，如`ServerSocketChannel`和`SocketChannel`
  * `FileChannel`：用于文件读写


##### 多路复用器Selector

* 多路复用器`Selector`是`Java NIO`编程的基础，熟练地掌握`Selector`对于`NIO`编程至关重要

* 多路复用器的功能：选择已经就绪的任务。
<br>即`Selector`会不断地轮询注册在其上的`Channel`，
<br>如果某个Channel上面发生读或者写事件，这个`Channel`就处于就绪状态，会被`Selector`轮询出来，
<br>然后通过SelectionKey就可以获取就绪的`Channel`集合，进行后续的`IO`操作。


* 一个`Selector`可以同时轮询多个`Channel`。
<br>由于JDK使用了epoll()代替传统的select，所以Selector没有最大连接句柄的限制，
<br>这意味着只需要一个线程负责Selector的轮询，就可以接入成千上万的客户端，这是一个巨大的进步。


##### NIO编程的优势
* 客户端发起的连接操作时异步的，可以在Selector中注册连接OP_CONNECT等待后续结果，
而不必像BIO一样被同步阻塞。
* SocketChannel的读写操作都是异步的，如果没有可读写的数据，它不会同步等待，直接返回，
这样IO通信线程就可以处理其他的链路，不需要同步等待这个链路可用。
* 线程模型的优化：同一个Selector可以同时处理成千上万个客户端连接，而且性能不会随着客户端的增加而
线性下降，因此非常适合做高性能、高负载的网络服务器。

但缺点就是：编码复杂，容易出错。

----

### 基于NIO2的AIO编程

`NIO2.0`引入了新的异步通道的概念，并提供了异步文件通道和异步套接字的实现。

异步通道可以通过两种方式来获取操作结果：

* 通过`java.util.concurrent.Future`类来表示异步操作的结果
* 在执行异步操作的时候提供一个`java.io.channels`

`CompletionHandler`接口的实现类作为操作完成的回调。

`NIO2.0` 的异步套接字通道是真正的异步非阻塞IO，对应于`UNIX`网络编程的事件驱动`IO（AIO）`，
它不需要通过多路复用器`Selector`对注册的通道进行轮询即可实现异步读写，从而简化了`NIO`的编程模型。

这种编程模型，逻辑更加清晰，更容易编写。

----

### 为什么要使用NIO编程

如果客户端并发连接数不多，周边对接的网元不多，服务器的负载也不重，那就完全没必要选择NIO做服务端；
如果是相反情况，则要综合考虑选择合适的NIO框架进行开发。


----

### 为什么要选择Netty

##### 不选择Java原生NIO编程的原因
* `NIO`的类库和`API`繁杂，使用麻烦，你需要熟练掌握`Selector、ServerSocketChannel、SocketChannel、ByteBuffer`等(个人感觉，AIO并不需要如此)
* 需要额外具备其他的额外技能做铺垫，如多线程编程
* 可靠性能力补齐，工作量和难度都非常大。例如客户端面临断连重试、网络闪断、半包读写、失败缓存、网络拥塞和异常码流的处理等问题，
`NIO`编程的特点是功能开发相对容易，但是可靠性能力补齐的工作量和难度都非常大。
* `JDK NIO`的`BUG`，例如`epoll bug`，它会导致`Selector`空轮询，最终导致`CPU` 100%。（不确定`JDK 1.8`及更高版本有没有修复）。

##### 为什么要选择Netty

`Netty`是业界最流行的`NIO`框架之一，它的健壮性、功能、性能、可定制性和可扩展性在同类框架中都是首屈一指的。

`Netty`已经经过了很多商用项目验证，例如`Hadoop`的`RPC`框架以及其他主流`RPC`框架底层都是使用`Netty`作为通信框架。

`Netty`的主要优点：

* `API`使用简单，开发门槛低；

* 功能强大，预置了多种编解码功能，支持多种主流协议；

* 定制能力强，可以通过`ChannelHandler`对通信框架进行灵活的扩展；

* 性能高，通过与其它业界主流的`NIO`框架对比，`Netty`的综合性能最优；

* 成熟、稳定，`Netty`修复了已经发现的所有`JDK NIO BUG`，业务开发人员不需要再为`NIO`的`BUG`而烦恼；

* 社区活跃，版本迭代周期短，发现的`BUG`可以被及时修复，同时，更多的新功能会被加入；

* 经历了大规模的商业应用考验，质量已经得到验证。在互联网、大数据、网络游戏、企业应用、电信软件等众多行业得到成功商用，证明了它可以完全满足不同行业的商业应用。

正是因为这些优点，`Netty`逐渐成为`Java NIO`编程的首选框架。

----

代码对应目录：

`time_server_bio`

`time_server_bio_using_threadpool`

`time_server_nio`

`time_server_aio`