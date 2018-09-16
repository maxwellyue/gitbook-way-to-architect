# Path和FIles

在Java1.7之前，Java提供了**`java.io.File`**来对文件进行操作，但是它却并不那么完美：

* 不会在平台中以一贯的方式来处理文件名
* 不支持高效文件属性访问
* 不允许复杂应用程序利用可用的文件系统特定特性（比如，符号链接）
* 大多数方法在出错时仅返回失败，而不会提供异常信息。

从Java 1.7开始，NIO2中提供了新的操作文件的方式：通过Files和Path。

## Path

**Path用于来表示文件路径或者文件：**A`Path`represents a path that is hierarchical and composed of a sequence of directory and file name elements separated by a special separator or delimiter。

**构造Path对象**

（1）使用Paths工具类的两个static方法

```java
Path path = Paths.get("C:/", "Xmp");
Path path2 = Paths.get("C:/Xmp");
        
URI u = URI.create("file:///C:/Xmp/dd");        
Path p = Paths.get(u);
```

（2）使用FileSystems

```java
Path path = FileSystems.getDefault().getPath("C:/", "access.log");
```

**File和Path之间的转换**

```java
File file = new File("C:/my.ini");
Path path = file.toPath();
File file1 = path.toFile();
```

## Files

**创建文件/目录：**Files.createFile\(path\)/Files.createDirectories\(path\)

```java
//创建文件
Path path = Paths.get("C:\\mystuff.txt");
try {
    if(!Files.exists(path)){
        Path file = Files.createFile(path);
    }
} catch (IOException e) {
    e.printStackTrace();
}
//创建目录
Path path = Paths.get("C:\\mystuff.txt");
try {
    if (!Files.exists(path)) {
        Files.createDirectories(path);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

**读取文件：**Files.newBufferedReader

```java
try {
    BufferedReader reader = Files.newBufferedReader(Paths.get("C:\\my.ini"), StandardCharsets.UTF_8);
    String str = null;
    while ((str = reader.readLine()) != null) {
        System.out.println(str);
    }
} catch (IOException e) {
    e.printStackTrace();
}
//Java 1.7 之前获取BufferedReader的方式
BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("C:\\my.ini"), Charset.forName("UTF-8")))
```

这里如果指定的字符编码不对，可能会抛出异常MalformedInputException，或者读取到乱码。

**写入：**Files.newBufferedWriter

```java
try {
    BufferedWriter writer = Files.newBufferedWriter(Paths.get("C:\\my2.ini"), StandardCharsets.UTF_8);
    writer.write("测试文件写操作");
    writer.flush();
    writer.close();
} catch (IOException e) {
    e.printStackTrace();
}
```

**复制**

```java
//从文件复制到文件
Files.copy(Path source, Path target, CopyOption options);

//从输入流复制到文件
Files.copy(InputStream in, Path target, CopyOption options);

//从文件复制到输出流
Files.copy(Path source, OutputStream out);
```

**获取/设置文件属性**

```java
//属性
FileTime lastModifiedTime = Files.getLastModifiedTime(path);
long size = Files.size(path);
boolean symbolicLink = Files.isSymbolicLink(path);
boolean directory = Files.isDirectory(path);
boolean sameFile = Files.isSameFile(path1, path2);
Map<String, Object> stringObjectMap = Files.readAttributes(path, "*");

//权限
Path path = Paths.get("/home/digdeep/.profile");
PosixFileAttributes attrs = Files.readAttributes(path, PosixFileAttributes.class);
Set<PosixFilePermission> posixPermissions = attrs.permissions();
posixPermissions.clear();
String owner = attrs.owner().getName();
String perms = PosixFilePermissions.toString(posixPermissions);

posixPermissions.add(PosixFilePermission.OWNER_READ);
posixPermissions.add(PosixFilePermission.GROUP_READ);
posixPermissions.add(PosixFilePermission.OTHERS_READ);
posixPermissions.add(PosixFilePermission.OWNER_WRITE);

Files.setPosixFilePermissions(path, posixPermissions);
```

**遍历一个文件夹：**Files.newDirectoryStream\(dir\)/Files.list\(path\)

```java
//方式1
Path dir = Paths.get("D:\\webworkspace");
try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir)) {
    for (Path e : stream) {
        System.out.println(e.getFileName());
    }
} catch (IOException e) {
    e.printStackTrace();
}
//方式2
try (Stream<Path> stream = Files.list(Paths.get("C:/"))) {
    Iterator<Path> ite = stream.iterator();
    while (ite.hasNext()) {
        Path pp = ite.next();
        System.out.println(pp.getFileName());
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

**递归遍历一个目录：**Files.walkFileTree

```java
Path startingDir = Paths.get("C:\\apache-tomcat-8.0.21");
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



### 参考

[文件系统 API](https://www.ibm.com/developerworks/cn/java/j-nio2-2/index.html)

[Java7新特性之文件操作](http://www.cnblogs.com/digdeep/p/4478734.html)





