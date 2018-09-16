### **构造**

如果文件不存在，则会抛出FileNotFoundException

* **File\(File parent, String child\) ：**根据 parent 抽象路径名和 child 路径名字符串创建一个新 File 实例。

* **File\(String pathname\) ：**通过将给定路径名字符串转换为抽象路径名来创建一个新 File 实例。

* **File\(String parent, String child\) ：**根据 parent 路径名字符串和 child 路径名字符串创建一个新 File 实例。

* **File\(URI uri\)： **通过将给定的 file: URI 转换为一个抽象路径名来创建一个新的 File 实例。

### **创建**

* **boolean createNewFile\(\) ：**当且仅当不存在具有此抽象路径名指定名称的文件时，不可分地创建一个新的空文件

* **boolean mkdir\(\)**：** **创建此抽象路径名指定的目录。

* **boolean mkdirs\(\) **：创建此抽象路径名指定的目录，包括所有必需但不存在的父目录。

### **删除**

* **boolean delete\(\) ：**删除文件或目录（只能是空文件夹），若文件不可删除，则抛出IOException。

* **void deleteOnExit\(\)： **在虚拟机终止时，请求删除此抽象路径名表示的文件或目录。

### **重命名**

* **boolean renameTo\(File dest\) ：**重新命名此抽象路径名表示的文件**。**

### **修改权限**

* **boolean setExecutable\(boolean executable\) ：**设置此抽象路径名所有者执行权限的一个便捷方法。

* **boolean setExecutable\(boolean executable, boolean ownerOnly\) **：设置此抽象路径名的所有者或所有用户的执行权限。

* **boolean setLastModified\(long time\) **：设置此抽象路径名指定的文件或目录的最后一次修改时间。

* **boolean setReadable\(boolean readable\) **：设置此抽象路径名所有者读权限的一个便捷方法。

* **boolean setReadable\(boolean readable, boolean ownerOnly\) **：设置此抽象路径名的所有者或所有用户的读权限**。**

* **boolean setReadOnly\(\) **：标记此抽象路径名指定的文件或目录，从而只能对其进行读操作。

* **boolean setWritable\(boolean writable\) **：设置此抽象路径名所有者写权限的一个便捷方法。

* **boolean setWritable\(boolean writable, boolean ownerOnly\) **：设置此抽象路径名的所有者或所有用户的写权限。

### **判断方法**

* **boolean canExecute\(\) ：** 测试应用程序是否可以执行此抽象路径名表示的文件。

* **boolean canRead\(\)**：** **测试应用程序是否可以读取此抽象路径名表示的文件。

* **boolean canWrite\(\) **：测试应用程序是否可以修改此抽象路径名表示的文件。

* **boolean exists\(\) **：测试此抽象路径名表示的文件或目录是否存在。

* **boolean isAbsolute\(\) **：测试此抽象路径名是否为绝对路径名。

* **boolean isDirectory\(\) **：测试此抽象路径名表示的文件是否是一个目录。

* **boolean isFile\(\) **：测试此抽象路径名表示的文件是否是一个标准文件。

* **boolean isHidden\(\) **：测试此抽象路径名指定的文件是否是一个隐藏文件。

### **获取方法**

* **File getAbsoluteFile\(\) **：**返回此抽象路径名的绝对路径名形式。**

* **String getAbsolutePath\(\) **：**返回此抽象路径名的绝对路径名字符串。**

* **File getCanonicalFile\(\) **：**返回此抽象路径名的规范形式。**

* **String getCanonicalPath\(\) **：**返回此抽象路径名的规范路径名字符串。**

* **long getFreeSpace\(\) **：**返回此抽象路径名指定的分区中未分配的字节数。**

* **String getName\(\) **：**返回由此抽象路径名表示的文件或目录的名称。**

* **String getParent\(\) **：**返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回 null。**

* **File getParentFile\(\) **：**返回此抽象路径名父目录的抽象路径名；如果此路径名没有指定父目录，则返回 null。**

* **String getPath\(\) **：**将此抽象路径名转换为一个路径名字符串。**

* **long getTotalSpace\(\) **：**返回此抽象路径名指定的分区大小。**

* **long getUsableSpace\(\) **：**返回此抽象路径名指定的分区上可用于此虚拟机的字节数。**

* **long lastModified\(\) **：**返回此抽象路径名表示的文件最后一次被修改的时间。**

* **long length\(\) **：**返回由此抽象路径名表示的文件的长度。**

* **static File\[\] listRoots\(\) **：**列出可用的文件系统根。**

* **String\[\] list\(\) **：**返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中的文件和目录。**

* **File\[\] listFiles\(\) **：**返回一个抽象路径名数组，这些路径名表示此抽象路径名表示的目录中的文件。**

## **过滤** {#8过滤文件}

* **String\[\] list\(FilenameFilter filter\) **：**返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中满足指定过滤器的文件和目录。**

* **File\[\] listFiles\(FileFilter filter\) **：**返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。**

* **File\[\] listFiles\(FilenameFilter filter\) **：**返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。**



内容来源：[Java----File类详解](https://blog.csdn.net/gfd54gd5f46/article/details/54918168)



