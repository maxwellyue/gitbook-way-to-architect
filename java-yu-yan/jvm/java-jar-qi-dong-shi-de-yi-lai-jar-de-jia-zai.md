# java -jar启动时的依赖jar的加载

当用`java -jar yourJarExe.jar`来运行一个经过打包的应用程序的时候，你会发现如何设置`-classpath`参数应用程序都找不到相应的第三方类，报`ClassNotFound`错误。实际上这是由于当使用`-jar`参数运行的时候，JVM会屏蔽所有的外部`classpath`，而只以本身`yourJarExe.jar`的内部`class`作为类的寻找范围。

## 解决方案

### BootStrap class扩展方案

Java 命令行提供了如何扩展bootStrap 级别class的简单方法：

```text
-Xbootclasspath:完全取代基本核心的Java class 搜索路径。不常用，否则要重新写所有Java 核心class。
-Xbootclasspath/a:追加在核心class搜索路径后面。常用
-Xbootclasspath/p:追加在核心class搜索路径前面。不常用,避免引起不必要的冲突。

//最常用的方式示例，如有多个，使用:(unix)或;(windows)进行分割
java -Xbootclasspath/a:/usrhome/thirdlib.jar: -jar yourJarExe.jar
```

###  extend class 扩展方案

`Java exten class` 存放在`$Java_home\jre\lib\ext`目录下。

解决的方案就是将所有要使用的第三方的jar包都复制到ext 目录下。

### User class扩展方案

当使用-jar执行可执行Jar包时，JVM将Jar包所在目录设置为classpath目录，所有的class搜索都在这个目录下开始。所以如果使用了其他第三方的jar包，一个比较可以接受的可配置方案就是利用jar包的Manifest扩展机制，步骤如下:

 1.将需要的第三方的jar包复制到你的可执行jar所在的目录或某个子目录下。

 2.修改Manifest 文件，在Manifest.mf文件里加入如下行

 `Class-Path:classes12.jar lib/thirdlib.jar`

 Class-Path 是可执行jar包运行依赖的关键词.详细内容可以参考 [http://java.sun.com/docs/books/tutorial/deployment/jar/downman.html](http://java.sun.com/docs/books/tutorial/deployment/jar/downman.html) 。要注意的是 Class-Path 只是作为你本地机器的CLASSPATH环境变量的一个缩写，也就是说用这个前缀表示在你的jar包执行机器上所有的CLASSPATH目录下寻找相应的第三方类/类库。你并不能通过 Class-Path 来加载位于你本身的jar包里面（或者网络上）的jar文件。因为从理论上来讲，你的jar发布包不应该再去包含其他的第三方类库（而应该通过使用说明来提醒用户去获取相应的支持类库）。如果由于特殊需要必须把其他的第三方类库（jar, zip, class等）直接打包在你自己的jar包里面一起发布，你就必须通过实现自定义的ClassLoader来按照自己的意图加载这些第三方类库。

然而，在实际开发中，对于java项目（非web项目），我们更常用`java class`的方式启动：

```text
java [options] classname [args]
```

具体操作：将所有依赖jar与你的项目jar（假设为example.jar，主类为`com.maxwell.example.Launcher`）放在同一目录\(比如为/xxx/yyy/lib\)，然后使用如下命令启动：

```text
java -classpath "/xxx/yyy/lib" com.maxwell.example.Launcher
```

### 参考

[-Xbootclasspath参数、java -jar参数运行应用时classpath的设置方法](http://www.cnblogs.com/duanxz/p/3482311.html)

