### Linux网络I/O模型

---

根据UNIX网络编程对I/O模型的分类，UNIX提供了5种I/O模型。

* 阻塞I/O模型
* 非阻塞I/O模型
* I/O复用模型
* 信号驱动I/O模型
* 异步I/O模型

Java NIO中的核心类库多路复用器Selector是基于epoll的多路复用技术实现。

> Linux中，epoll是事件驱动，而select/poll则是顺序扫描，且支持的fd（文件描述符）有限。

epoll是Linux下的一种IO多路复用技术，可以非常高效的处理数以百万计的socket句柄。

先看看使用c封装的3个epoll系统调用：

* int epoll\_create\(int size\)
  epoll\_create建立一个epoll对象。参数size是内核保证能够正确处理的最大句柄数，多于这个最大数时内核可不保证效果。
* int epoll\_ctl\(int epfd, int op, int fd, struct epoll\_event \*event\)
  epoll\_ctl可以操作epoll\_create创建的epoll，如将socket句柄加入到epoll中让其监控，或把epoll正在监控的某个socket句柄移出epoll。
* int epoll\_wait\(int epfd, struct epoll\_event \*events,int maxevents, int timeout\)
  epoll\_wait在调用时，在给定的timeout时间内，所监控的句柄中有事件发生时，就返回用户态的进程。

epoll初始化时，会向内核注册一个文件系统，用于存储被监控的句柄文件，调用epoll\_create时，会在这个文件系统中创建一个file节点。同时epoll会开辟自己的内核高速缓存区，以红黑树的结构保存句柄，以支持快速的查找、插入、删除。还会再建立一个list链表，用于存储准备就绪的事件。

当执行epoll\_ctl时，除了把socket句柄放到epoll文件系统里file对象对应的红黑树上之外，还会给内核中断处理程序注册一个回调函数，告诉内核，如果这个句柄的中断到了，就把它放到准备就绪list链表里。所以，当一个socket上有数据到了，内核在把网卡上的数据copy到内核中后，就把socket插入到就绪链表里。

当epoll\_wait调用时，仅仅观察就绪链表里有没有数据，如果有数据就返回，否则就sleep，超时时立刻返回。

epoll的两种工作模式：

* LT：level-trigger，水平触发模式，只要某个socket处于readable/writable状态，无论什么时候进行epoll\_wait都会返回该socket。
* ET：edge-trigger，边缘触发模式，只有某个socket从unreadable变为readable或从unwritable变为writable时，epoll\_wait才会返回该socket。

### Linix中的IO多路复用技术

---

IO多路复用技术是指：将多个IO的阻塞复用到同一个select的阻塞上，从而使系统在单线程的情况下，可以同时处理多个客户端请求。

epoll相比select作了很多重大改进：

* 支持一个进程打开的socket fd 不受限制，仅受限于操作系统的最大文件句柄数
* IO效率不会随fd数目的增加而线性下降（epoll实现了伪AIO）
* 使用mmap加速内核与用户空间的消息传递
* API更加简单

### Java 的IO演进

---

###### JDK1.0~JDK1.3：BIO

所有Socket通信都采用了同步阻塞模式（BIO），这种一请求一应答的通信模型简化了应用开发，但牺牲了性能。

###### JDK1.4：NIO

提供了很多进行异步IO的API和类库

* ByteBuffer：进行异步IO操作的缓冲区
* Pipe：进行异步IO操作的管道
* Channel：如ServerSocketChannel和SocketChannel
* 多种字符集的编码和解码
* 多路复用器selector
* 文件通道FileChannel

但对文件的处理能力不足：

* 没有统一的文件属性（例如读写权限）
* API能力较弱，比如目录的级联创建和递归遍历，要自己实现
* 所有的文件操作都是同步阻塞调用，不支持异步文件的读写操作
* 底层存储系统的一些高级API无法使用

###### JDK1.7：NIO2.0

相比NIO，改进有：

* 能够提供批量获取文件属性的API
* 提供了AIO功能
* 提供了通道功能，包括对配置和多播数据报的支持等



