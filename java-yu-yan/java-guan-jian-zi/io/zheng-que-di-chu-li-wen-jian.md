# 处理文件的技巧

**拼接文件路径：**使用Path，且文件分隔符使用File.pathSeparator，不要硬编码为`/`或者`\`

```java
//假设有如下文件：/Users/yue/Desktop/test.ev4
Path path = Paths.get("/Users", "yue", "Desktop", "test.ev4");

//注意，这种方式是错误的，即Path只会从第二个参数开始，自动添加文件分隔符，且第一个参数应该是有效路径
Path path = Paths.get(File.pathSeparator, "Users", "yue", "Desktop", "test.ev4");
```

**读取文件所有内容前，先判断文件大小，防止OOM**

```java
public static byte[] readAllBytes(String fileName, long maxSize) throws IOException {
    Path path = Paths.get(fileName);
    long size = Files.size(path);
    if (size > maxSize) {
        throw new IOException("file: " + path + ", size:" + size + "> " + maxSize);
    }
    return Files.readAllBytes(path);
}

public static List<String> readAllLines(String fileName, Charset charset, long maxSize) throws IOException {
    Path path = Paths.get(fileName);
    long size = Files.size(path);
    if (size > maxSize) {
        throw new IOException("file: " + path + ", size:" + size + "> " + maxSize);
    }
    return Files.readAllLines(path, charset);
}
```

**遍历文件：**使用FileVisitor

```java
Path startingDir = Paths.get("/Users/yue/Desktop");
try {
    Files.walkFileTree(startingDir, new SimpleFileVisitor<Path>() {
        @Override
        public FileVisitResult visitFile(Path path, BasicFileAttributes attrs) throws IOException {
            System.out.println(path.getFileName());
            return FileVisitResult.CONTINUE;
        }
    });
} catch (IOException e) {
    e.printStackTrace();
}
```

**判断文件是否在某路径下：**使用file.getCanonicalPath\(\)，不必递归

```java
//使用file.getCanonicalPath()
public static boolean isSubFile(File dir, File file) throws IOException {
    return file.getCanonicalPath().startsWith(dir.getCanonicalPath() + File.pathSeparator);
}
//递归方式
public static boolean isInDirectory(File dir, File file) {

    if (file == null)
        return false;

    if (file.equals(dir))
        return true;

    return isInDirectory(dir, file.getParentFile());
}
```

**监视文件改变：**使用WatchService

```java
public class Configs {
    public static Properties properties;
    static {
        final String configFilename = "app.properties";
        final ClassPathResource resource = new ClassPathResource(configFilename);
        try {
            //先加载一次配置文件
            properties = PropertiesLoaderUtils.loadProperties(resource);
            //设置监听配置文件所在目录的文件删除/新增操作
            final WatchService watcher = FileSystems.getDefault().newWatchService();
            Paths.get(resource.getFile().getParent()).register(watcher,
                    StandardWatchEventKinds.ENTRY_CREATE,
                    StandardWatchEventKinds.ENTRY_DELETE,
                    StandardWatchEventKinds.ENTRY_MODIFY);
            //开启监听线程
            Thread watchThread = new Thread(() -> {
                try {
                    WatchKey watchKey = watcher.take();
                    for (WatchEvent<?> event : watchKey.pollEvents()) {
                        //如果是配置文件发生变化，则重新加载配置文件
                        System.out.println(event.kind() + ":::" + event.context().toString());
                        if (Objects.equals(configFilename, event.context().toString())) {
                            properties = PropertiesLoaderUtils.loadProperties(resource);
                            break;
                        }
                    }
                    watchKey.reset();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            });
            watchThread.setDaemon(true);
            watchThread.start();

            //JVM停止时，关闭watcher线程
            Runtime.getRuntime().addShutdownHook(new Thread(() -> {
                try {
                    watcher.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static boolean getBoolean(String key) {
        String property = properties.getProperty(key, "false");
        return "true".equals(property);
    }
    public static int getInt(String key) {
        String property = properties.getProperty(key, "0");
        return Integer.valueOf(property);
    }
}
```

**Web服务器防止非法的文件路径访问**

字符截断攻击和文件历遍漏洞原理：在文件名中插入%00的URL编码，web服务器会把%00后面的内容抛弃。

例如这样的URL：http://www.test.com/../../../../etc/passwd%00.gif

防范方法

* 写入文件前，判断文件是否在父路径下，参考上面的函数。
* 利用Java的安全机制

```
// All files in /img/java can be read
grant codeBase "file:/home/programpath/" {
  permission java.io.FilePermission "/img/java", "read";
};
```

**参考**

[在Java里处理文件的技巧](http://www.importnew.com/19413.html)：大部分内容来源于此，略有修正和补充

[Check if file is in \(sub\)directory](https://stackoverflow.com/questions/18227634/check-if-file-is-in-subdirectory)



