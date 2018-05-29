# Stream API详解

## 概述

当我们使用一个流的时候，通常包括三个基本步骤：

①获取一个数据源（source）→  ②数据转换 → ③执行操作，获取想要的结果。每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道，如下图所示。

![&#x56FE; 1.  &#x6D41;&#x7BA1;&#x9053; \(Stream Pipeline\) &#x7684;&#x6784;&#x6210;](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/img001.png)

1、**生成 Stream Source的方式**

①从 Collection 和数组

```java
Collection.stream()   
Collection.parallelStream()
Arrays.stream(T array) or Stream.of()

//演示代码
List<String> list = Arrays.asList("dog", "cat", "pig", "rabbit");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream();
int[] array = {1, 2, 34, 5, 56, 4, 24};
IntStream intStream = Arrays.stream(array);
Stream<Integer> integerStream = Stream.of(1, 2, 34, 5, 56, 4, 24);
```

②从 BufferedReader

```java
java.io.BufferedReader.lines()
```

③静态工厂

```java
java.util.stream.IntStream.range()
java.nio.file.Files.walk()
```

④自己构建

```java
java.util.Spliterator
```

⑤其它

```java
Random.ints()
BitSet.stream()
Pattern.splitAsStream(java.lang.CharSequence)
JarFile.stream()
```

2、**流的操作类型**

* **Intermediate（中间操纵）**：一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。**这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。**
* **Terminal（终端操作）**：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。

在对于一个 Stream 进行多次转换操作 \(Intermediate 操作\)，每次都对 Stream 的每个元素进行转换，而且是执行多次，这样时间复杂度就是 N（转换次数）个 for 循环里把所有操作都做掉的总和吗？其实不是这样的，转换操作都是 lazy 的，多个转换操作只会在 Terminal 操作的时候融合起来，一次循环完成。我们可以这样简单的理解，Stream 里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在 Terminal 操作的时候循环 Stream 对应的集合，然后对每个元素执行所有的函数。

还有一种操作被称为 **short-circuiting**，用以指：

* 对于一个 intermediate 操作，如果它接受的是一个无限大（infinite/unbounded）的 Stream，但返回一个有限的新 Stream。
* 对于一个 terminal 操作，如果它接受的是一个无限大的 Stream，但能在有限的时间计算出结果。

当操作一个无限大的 Stream，而又希望在有限时间内完成操作，则在管道内拥有一个 short-circuiting 操作是必要非充分条件。

一个流操作的示例

```java
int sum = widgets.stream()                //获取当前小物件的 source
        .filter(w -> w.getColor() == RED) //intermediate 操作，进行数据筛选和转换
        .mapToInt(w -> w.getWeight())     //intermediate 操作，进行数据筛选和转换
        .sum();                           //terminal 操作，对符合条件的全部小物件作重量求和
```

## 流使用详解







## 参考

[五分钟学习Java8的流编程](https://juejin.im/post/5b07f4536fb9a07ac90da4e5?utm_source=gold_browser_extension)

[Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/)

