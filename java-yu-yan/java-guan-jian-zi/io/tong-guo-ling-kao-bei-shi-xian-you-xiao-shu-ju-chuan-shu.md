# 零拷贝实现数据传输

## **什么是零拷贝**

考虑这样一种场景：用户发送下载一个文件的请求，服务器收到请求后，将本地磁盘的一个文件原封不动地返回给用户。整个处理过程如下：

![](/assets/io-traditional.png)

* 第1步：应用程序向系统发送要读取文件的指令（read\(\)），此时上下文由user切换到kernel。系统收到指令后，调用sys\_read\(\) 来从文件中读取data（第一次copy），存储在kernel的buffer中。
* 第2步：data数据从kernel的buffer中被copy到user buffer中，此时read\(\)成功返回，上下文从kernel切换到user。至此，数据存储在user的buffer中。
* 第3步：应用程序向系统发送要发送数据的指令（send\(\)）， 此时上下文由user切换到kernel。系统收到指令后，将data从user的buffer中copy到kernel buffer中。当然，这次的kernel buffer和第一步的buffer是不同的buffer。
* 第4步：send\(\) 方法返回，此时上下文从kernel切换到user。同时，第四次copy发生，DMA将data从kernel buffer拷贝到protocol engine中。

整个过程中，共发送了4次上下文切换，并将文件数据copy了4次。其中，第1步和第4步的copy由DMA（data memory access）完成，第2步和第3步的copy由CPU完成。

在这个场景中，应用程序实际上只是作为一种低效的中间介质，用来把disk file的data传给socket。假如将这些中间操作都省去，直接把disk的data传输给socket，效率将会大大提高。这就是**所谓的零拷贝技术：减少context switch，消除和CPU有关的数据拷贝。**

使用零拷贝技术，user层面的使用方法没有变，但是内部原理却发生了变化（Linux 内核 2.4 及后期版本中）：

1. transferTo\(\)方法使得文件内容被copy到了kernel buffer，这一动作由DMA engine完成。
2. 没有data被copy到socket buffer。取而代之的是socket buffer被追加了一些descriptor的信息，包括data的位置和长度。然后DMA engine直接把data从kernel buffer传输到protocol engine，这样就消除了需要占用CPU的拷贝操作。

![](/assets/zero-copy.png)

在上面的场景中，使用零拷贝技术后，消除了CPU来copy的操作（2次），只剩下了DMA的2次copy操作，上下文只需要切换2次。

## Java中使用零拷贝

Java 的零拷贝多在网络应用程序中使用。关键的API是`java.nio.channel.FileChannel`的`transferTo()/transferFrom()`方法。我们可以用这两个方法来把bytes直接从调用它的channel传输到另一个writable byte channel，中间不会使data经过应用程序。

**拷贝文件的示例**

```java
@Test
public void testFileZeroCopy() throws IOException {
    FileChannel fromChannel = new RandomAccessFile("/Users/yue/Desktop/test.ev4", "rw").getChannel();
    FileChannel toChannel = new RandomAccessFile("/Users/yue/Desktop/copy.ev4", "rw").getChannel();

    fromChannel.transferTo(0, fromChannel.size(), toChannel);
    fromChannel.close();
    toChannel.close();
}
```

**通过网络把一个文件从client传到server示例**

```java
package com.maxwell.learning.common.io;

/**
 * @author yuezengcun <yuezengcun@meicai.cn>
 * @since
 */

import java.io.FileInputStream;
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.SocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;

/**
 * disk-nic零拷贝
 */
class ZeroCopyServer {

    ServerSocketChannel listener = null;

    public void start() throws IOException {
        InetSocketAddress listenAddress = new InetSocketAddress(8080);
        listener = ServerSocketChannel.open();
        ServerSocket ss = listener.socket();
        ss.setReuseAddress(true);
        ss.bind(listenAddress);
        System.out.println("监听端口:" + listenAddress.toString());
    }

    public static void main(String[] args) throws IOException {
        ZeroCopyServer dns = new ZeroCopyServer();
        dns.start();
        dns.receive();
    }

    private void receive() {
        ByteBuffer dst = ByteBuffer.allocate(1024 * 4);
        try {
            while (true) {
                SocketChannel conn = listener.accept();
                System.out.println("创建连接: " + conn);
                conn.configureBlocking(true);
                int nread = 0;
                while (nread != -1) {
                    try {
                        nread = conn.read(dst);
                    } catch (IOException e) {
                        e.printStackTrace();
                        nread = -1;
                    }
                    dst.rewind();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

class ZeroCopyClient {

    public static void main(String[] args) throws IOException {
        SocketAddress server = new InetSocketAddress("localhost", 8080);

        SocketChannel client = SocketChannel.open();
        client.connect(server);
        client.configureBlocking(true);

        FileChannel fileChannel = new FileInputStream("/Users/yue/Desktop/test.ev4").getChannel();
        //零拷贝发送文件
        fileChannel.transferTo(0, fileChannel.size(), client);

        client.close();
        fileChannel.close();
    }
}
```

## 参考

[通过零拷贝实现有效数据传输](https://www.ibm.com/developerworks/cn/java/j-zerocopy/)

[JAVA Zero Copy的相关知识](https://my.oschina.net/cloudcoder/blog/299944)

  


