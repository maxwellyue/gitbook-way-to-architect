# **RandomAccessFile**

## 概述

RandomAccessFile 是随机访问文件\(包括读/写\)的类。它支持对文件随机访问的读取和写入：我们可以从指定的位置读取/写入文件数据。它在java.io包中是一个特殊的类，既不是输入流也不是输出流，但它既可以读文件，也可以写文件：实现了DataInput和 DataOut接口。

```java
public class RandomAccessFile implements DataOutput, DataInput, Closeable {}
```

**什么是随机访问**

对于流而言，有几个限制：

* 要么读，要么写，不能同时读和写
* 只能从头读到尾，且不能重复读，虽然通过缓冲可以实现部分重读，但是有限制

而随机访问文件的含义是指，给定一个文件，可以从文件的任意一个位置进行读取/写入。比如我们要给一个4G的大文件追加一行内容，不可能将其全部加载到内存，而只是需要找到文件的末尾，然后在末尾进行追加。

**打开文件的模式**

RandomAccessFile共有4种模式："r", "rw", "rws"和"rwd"。

* "r" ：以只读方式打开。调用结果对象的任何 write 方法都将导致抛出 IOException
* "rw" ：打开以便读取和写入
* "rws"： 打开以便读取和写入。相对于 "rw"，"rws" 还要求对“文件的内容”或“元数据”的每个更新都同步写入到基础存储设备
* "rwd"：打开以便读取和写入，相对于 "rw"，"rwd" 还要求对“文件的内容”的每个更新都同步写入到基础存储设备

**如何随机访问**

RandomAccessFile内部有一个文件指针，指向当前读写的位置，各种read/write操作都会自动更新该指针，与流不同的是，RandomAccessFile可以获取该指针，也可以更改该指针，相关方法是：

```java
//返回文件指针所在的位置
public native long getFilePointer() throws IOException;

//将文件指针定位到pos的位置
public void seek(long pos) throws IOException;
```

RandomAccessFile是通过本地方法，最终调用操作系统的API来实现文件指针调整的。

InputStream有一个skip方法，可以跳过输入流中n个字节，默认情况下，它是通过实际读取n个字节实现的，RandomAccessFile有一个类似方法，不过它是通过更改文件指针实现的：

```java
public int skipBytes(int n) throws IOException
```

RandomAccessFile可以直接获取文件长度，返回文件字节数，方法为：

```java
public native long length() throws IOException;
```

它还可以直接修改文件长度，方法为：

```java
public native void setLength(long newLength) throws IOException;
```

如果当前文件的长度小于newLength，则文件会扩展，扩展部分的内容未定义。如果当前文件的长度大于newLength，则文件会收缩，多出的部分会截取，如果当前文件指针比newLength大，则调用后会变为newLength。

此外，RandomAccessFile中有如下方法尽量不要使用：

```java
public final void writeBytes(String s) throws IOException
public final String readLine() throws IOException
```

看上去，writeBytes可以直接写入字符串，而readLine可以按行读入字符串，实际上，这两个方法都是有问题的，它们都没有编码的概念，都假定一个字节就代表一个字符，这对于中文显然是不成立的，所以，应避免使用这两个方法。

## 使用示例

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

## 



参考

[计算机程序的思维逻辑 \(60\) - 随机读写文件及其应用](https://juejin.im/post/586267d761ff4b006cf136d9)

[java io系列26之 RandomAccessFile](https://www.cnblogs.com/skywang12345/p/io_26.html)



