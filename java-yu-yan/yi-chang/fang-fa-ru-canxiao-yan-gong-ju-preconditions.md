### 为什么需要Preconditions

在日常开发中，我们经常会对方法的输入参数做一些数据格式上的验证，以便保证方法能够按照正常流程执行下去。对于可预知的一些数据上的错误，我们一定要做事前检测和判断，来避免程序流程出错，而不是完全通过错误处理来保证流程正确执行，毕竟错误处理是比较消耗资源的方式，并且谨慎编程所持有的一个态度就是：**永远不要相信收到的数据是合法的。**

平常写代码时我们可能会采用如下方式进行参数判断：

```java
private void add(boolean bool, int[] array, int position) {
    if (!bool) {
        throw new IllegalArgumentException("preCondition not allow!!");
    }
    if (array == null) {
        throw new NullPointerException("array is null!!");
    }
    if (array.length == 0) {
        throw new IllegalArgumentException("array length is 0!!");
    }
    if (position > array.length || position < 0) {
        throw new ArrayIndexOutOfBoundsException("position error!!");
    }

    //ok, do something...
}
```

采用这种方式，参数的判断都需要自己来逐个写方法判断，代码量多并且无法复用。

### Preconditions

Preconditions是Guava类库中提供了的一个作参数检查的工具类， 该类可以大大地简化我们代码中对于参数的预判断和处理，让我们对方法输入参数的验证实现起来更加简单。

下面，使用Preconditions对上述代码进行改造：

```java
private void testPreconditions(boolean bool, int[] array, int position) {
   Preconditions.checkArgument(bool);
   Preconditions.checkNotNull(array);
   Preconditions.checkElementIndex(position, array.length);
   //ok, do something...
}
```

还可以自定义错误信息：

```java
private static void testPreconditions(boolean bool, int[] array, int position) {
    Preconditions.checkArgument(bool, "not allow!!");
    Preconditions.checkNotNull(array, "array is null!!");
    Preconditions.checkElementIndex(position, array.length, "position error!!");
    //ok, do something...
}
```



**参考**

[Google Guava中Preconditions的用法，让前置条件判断变得更优雅](https://blog.csdn.net/zivensonice/article/details/51912188)

[Guava学习笔记：Preconditions优雅的检验参数](https://www.cnblogs.com/peida/p/Guava_Preconditions.html)

