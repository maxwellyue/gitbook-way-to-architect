## 基本概念

所谓内存映射文件，就是将文件映射到内存，文件对应于内存中的一个字节数组，对文件的操作变为对这个字节数组的操作，而字节数组的操作直接映射到文件上。这种映射可以是映射文件全部区域，也可以是只映射一部分区域。

不过，这种映射是操作系统提供的一种假象，文件一般不会马上加载到内存，操作系统只是记录下了这回事，当实际发生读写时，才会按需加载。操作系统一般是按页加载的，页可以理解为就是一块，页的大小与操作系统和硬件相关，典型的配置可能是4K, 8K等，当操作系统发现读写区域不在内存时，就会加载该区域对应的一个页到内存。

这种按需加载的方式，使得内存映射文件可以方便处理非常大的文件，内存放不下整个文件也不要紧，操作系统会自动进行处理，将需要的内容读到内存，将修改的内容保存到硬盘，将不再使用的内存释放。

在应用程序写的时候，它写的是内存中的字节数组，这个内容什么时候同步到文件上呢？这个时机是不确定的，由操作系统决定，不过，只要操作系统不崩溃，操作系统会保证同步到文件上，即使映射这个文件的应用程序已经退出了。

在一般的文件读写中，会有两次数据拷贝，一次是从硬盘拷贝到操作系统内核，另一次是从操作系统内核拷贝到用户态的应用程序。而在内存映射文件中，一般情况下，只有一次拷贝，且内存分配在操作系统内核，应用程序访问的就是操作系统的内核内存空间，这显然要比普通的读写效率更高。

内存映射文件的另一个重要特点是，它可以被多个不同的应用程序共享，多个程序可以映射同一个文件，映射到同一块内存区域，一个程序对内存的修改，可以让其他程序也看到，这使得它特别适合用于不同应用程序之间的通信。

操作系统自身在加载可执行文件的时候，一般都利用了内存映射文件，比如：

* 按需加载代码，只有当前运行的代码在内存，其他暂时用不到的代码还在硬盘
* 同时启动多次同一个可执行文件，文件代码在内存也只有一份
* 不同应用程序共享的动态链接库代码在内存也只有一份

内存映射文件也有局限性，比如，它不太适合处理小文件，它是按页分配内存的，对于小文件，会浪费空间，另外，映射文件要消耗一定的操作系统资源，初始化比较慢。

简单总结下，对于一般的文件读写不需要使用内存映射文件，但如果处理的是大文件，要求极高的读写效率，比如数据库系统，或者需要在不同程序间进行共享和通信，那就可以考虑内存映射文件。

理解了内存映射文件的基本概念，接下来，我们看怎么在Java中使用它。

## 用法

### 映射文件

内存映射文件需要通过FileInputStream/FileOutputStream或RandomAccessFile，它们都有一个方法：

```java
public FileChannel getChannel()
```

FileChannel有如下方法：

```java
public MappedByteBuffer map(MapMode mode, long position, long size) throws IOException
```

map方法将当前文件映射到内存，映射的结果就是一个MappedByteBuffer对象，它代表内存中的字节数组，待会我们再来详细看它。map有三个参数，mode表示映射模式，positon表示映射的起始位置，size表示长度。

mode有三个取值：

* MapMode.READ\_ONLY：只读
* MapMode.READ\_WRITE：既读也写
* MapMode.PRIVATE：私有模式，更改不反映到文件，也不被其他程序看到

这个模式受限于背后的流或RandomAccessFile，比如，对于FileInputStream，或者RandomAccessFile但打开模式是"r"，那mode就不能设为MapMode.READ\_WRITE，否则会抛出异常。

如果映射的区域超过了现有文件的范围，则文件会自动扩展，扩展出的区域字节内容为0。

映射完成后，文件就可以关闭了，后续对文件的读写可以通过MappedByteBuffer。

看段代码，比如以读写模式映射文件"abc.dat"，代码可以为：

```java
RandomAccessFile file = new RandomAccessFile("abc.dat","rw");
try {
    MappedByteBuffer buf = file.getChannel().map(MapMode.READ_WRITE, 0, file.length());
    //使用buf...
} catch (IOException e) {
    e.printStackTrace();
}finally{
    file.close();
}
```

### MappedByteBuffer

MappedByteBuffer是ByteBuffer的子类，而ByteBuffer是Buffer的子类。ByteBuffer和Buffer不只是给内存映射文件提供的，它们是Java NIO中操作数据的一种方式。ByteBuffer可以简单理解为就是封装了一个字节数组，这个字节数组的长度是不可变的，在内存映射文件中，这个长度由map方法中的参数size决定。

除了继承自ByteBuffer的方法，MappedByteBuffer中还定义了一些方法：

```java
//检查文件内容是否真实加载到了内存，这个值是一个参考值，不一定精确
public final boolean isLoaded()

//尽量将文件内容加载到内存
public final MappedByteBuffer load()

//将对内存的修改强制同步到硬盘上
public final MappedByteBuffer force()
```







内容来源：[计算机程序的思维逻辑 \(61\) - 内存映射文件及其应用](https://juejin.im/post/58626ac361ff4b006cf14faf)

