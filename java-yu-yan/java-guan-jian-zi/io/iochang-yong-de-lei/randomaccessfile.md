# **RandomAccessFile**

RandomAccessFile 是随机访问文件\(包括读/写\)的类。它支持对文件随机访问的读取和写入：我们可以从指定的位置读取/写入文件数据。它在java.io包中是一个特殊的类，既不是输入流也不是输出流，但它既可以读文件，也可以写文件：实现了DataInput和 DataOut接口。

```java
public class RandomAccessFile implements DataOutput, DataInput, Closeable {}
```

**什么是随机访问**

随机访问文件的含义是指，给定一个文件，可以从文件的任意一个位置进行读取/写入。比如我们要给一个4G的大文件追加一行内容，不可能将其全部加载到内存，而只是需要找到文件的末尾，然后在末尾进行追加。

**RandomAccessFile中常用的方法**

```java
//返回文件记录指针所在的位置
public native long getFilePointer() throws IOException;

//将文件记录指针定位到pos的位置
public void seek(long pos) throws IOException;
```

使用示例1：从任意位置读

```java
@Test
public void testRandomAccessFile() throws IOException {
    RandomAccessFile raf = new RandomAccessFile("xxx.rr", "rw");
    //写入10个字符，占用20个字节
    raf.writeChars("aaaaabbbbb");
    raf = new RandomAccessFile("xxx.rr", "rw");
    //定位到第10个字节，读的时候会从第11个字节开始读
    raf.seek(10);
    for (int i = 0; i < 5; i++) {
        char c = raf.readChar();
        System.out.println(String.valueOf(c));
    }
    raf.close();
}
//输出如下
b
b
b
b
b
```

使用示例2：追加内容

TODO







## 打开文件的模式

RandomAccessFile共有4种模式："r", "rw", "rws"和"rwd"。

* "r" ：以只读方式打开。调用结果对象的任何 write 方法都将导致抛出 IOException
* "rw" ：打开以便读取和写入
* "rws"： 打开以便读取和写入。相对于 "rw"，"rws" 还要求对“文件的内容”或“元数据”的每个更新都同步写入到基础存储设备
* "rwd"：打开以便读取和写入，相对于 "rw"，"rwd" 还要求对“文件的内容”的每个更新都同步写入到基础存储设备

参考

[java io系列26之 RandomAccessFile](https://www.cnblogs.com/skywang12345/p/io_26.html)

