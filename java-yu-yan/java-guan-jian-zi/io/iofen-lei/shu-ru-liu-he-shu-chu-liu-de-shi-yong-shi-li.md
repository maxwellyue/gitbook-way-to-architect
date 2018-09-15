## 规则

---

### 关于**flush**

~~对于输出流而言，如果其具有Buffer功能，则每次的write操作其实是将内容保存在内存中的数组中（称之为buffer），当buffer容量到达上限时，会自动将buffer的数据写入到目的地；假如buffer没有到达上限，通过手动调用flush\(\)方法，也会将buffer的数据写入到目的地。~~

> ~~对于输入流，则没有这个概念，只有OutputStream/Writer/Console实现了Flushable接口。~~

* ~~FileWriter：内部通过StreamEncoder将字符流编码为字节流，StreamEncoder的默认的的缓冲区大小为8192byte~~
* ~~FileOutputStream：没有缓冲区~~
* ~~BufferedOutputStream：buffer为字节数组，默认的的缓冲区大小为8192byte~~

* ~~BufferedWriter：buffer为字符数组，默认的的缓冲区大小为8192char，即2\*8192byte（一个char占用2个byte）~~

~~什么时候需要手动调用flush\(\)~~

~~使用输出流将内容（假设为字符串）写入到目的地（假设为文件），在write\(\)之后（没有close这个输出流之前），假如使用了这个文件，那么极有可能这个文件中实际的字符与内容不相同（除非这个字符串的大小恰好为8\*1024byte的整数倍）。所以，这种情况需要手动调用flush\(\)方法。但是，如果只是在close这个输出流之后，才去使用输出流的输出目的的内容，则不必手动调用flush方法，因为close\(\)方法中会自动调用flush\(\)（API docs no guarantee，but most ），以确保关闭流之前将内容全部写入。~~

查了很多资料，没有找到确切的说法。

## File

---

对于文本文件，我们可以通过FileReader或者FileWriter进行字符的读写，但是对于一些图片或者视频这类文件，显然字符是无意义的，需要使用FileInputStream与FileOutputStream等字节流进行操作。

示例1：在硬盘上，创建一个文件并写入一些文字数据

```java
@Test
public void testFileWriter() {
    try (FileWriter fw = new FileWriter("demo.txt");) {
        fw.write("abcde");
        fw.flush();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

@Test
public void testFileReader() {
    try (FileReader fileReader = new FileReader("demo.txt");) {
        int ch = 0;
        while ((ch = fileReader.read()) != -1) {
            System.out.println("ch=" + (char) ch);
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

@Test
public void testFileReader2() {
    try (FileReader fileReader = new FileReader("demo.txt");) {
        char[] buf = new char[1024];
        int num = 0;
        while ((num = fileReader.read(buf)) != -1) {
            System.out.println(new String(buf, 0, num));
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

示例2：复制图片：对于输入流，要去文件必须已经存在；对于输出流，文件如果不存在，则会自动创建

```java
@Test
public void testFileStream() {
    try (OutputStream outputStream = new FileOutputStream("/Users/yue/Desktop/copy.png");
         InputStream inputStream = new FileInputStream("/Users/yue/Desktop/test.png");
    ) {
        byte[] buffer = new byte[1024 * 1024];
        while (inputStream.read(buffer) != -1) {
            outputStream.write(buffer);
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

示例3：复制视频：使用缓冲功能

```java
@Test
public void testBufferedStream() {
    long start = System.currentTimeMillis();
    try (
            InputStream inputStream = new FileInputStream("/Users/yue/Desktop/test.ev4");
            InputStream bis = new BufferedInputStream(inputStream, 1024 * 1024);
            OutputStream outputStream = new FileOutputStream("/Users/yue/Desktop/copy.ev4");
            OutputStream bos = new BufferedOutputStream(outputStream, 1024 * 1024);
    ) {
        byte[] buffer = new byte[1024 * 1024];
        while ((bis.read(buffer)) != -1) {
            bos.write(buffer);
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }

    long end = System.currentTimeMillis();
    System.out.println("使用缓冲，耗时：" + (end - start));
}
```











[Using flush\(\) before close\(\)](https://stackoverflow.com/questions/9858495/using-flush-before-close)



