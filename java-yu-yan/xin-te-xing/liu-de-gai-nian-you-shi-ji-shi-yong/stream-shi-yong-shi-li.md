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

简单说，对 Stream 的使用就是实现一个 filter-map-reduce 过程，产生一个最终结果，或者导致一个副作用（side effect）。

#### 流的构造与转换 {#major7}

下面提供最常见的几种构造 Stream 的样例。

构造流的几种常见方法

```java
// 1. Individual values
Stream stream = Stream.of("a", "b", "c");
// 2. Arrays
String[] strArray = new String[]{"a", "b", "c"};
stream = Stream.of(strArray);
stream = Arrays.stream(strArray);
// 3. Collections
List<String> list = Arrays.asList(strArray);
stream = list.stream();
```

 需要注意的是，对于基本数值型，目前有三种对应的包装类型 Stream：`IntStream、LongStream、DoubleStream`。当然我们也可以用 `Stream<Integer>、Stream<Long> >、Stream<Double>`，但是 `boxing` 和 `unboxing` 会很耗时，所以特别为这三种基本数值型提供了对应的 Stream。Java 8 中还没有提供其它数值型 Stream，因为这将导致扩增的内容较多。而常规的数值型聚合运算可以通过上面三种 Stream 进行。

数值流的构造

```java
IntStream.of(new int[]{1, 2, 3}).forEach(System.out::println);
IntStream.range(1, 3).forEach(System.out::println);
IntStream.rangeClosed(1, 3).forEach(System.out::println);
```

 流转换为其它数据结构

```java
// 1. Array
String[] strArray1 = stream.toArray(String[]::new);
// 2. Collection
List<String> list1 = stream.collect(Collectors.toList());
List<String> list2 = stream.collect(Collectors.toCollection(ArrayList::new));
Set set1 = stream.collect(Collectors.toSet());
Stack stack1 = stream.collect(Collectors.toCollection(Stack::new));
// 3. String
String str = stream.collect(Collectors.joining()).toString();
```

 注意：一个 Stream 只可以使用一次，上面的代码为了简洁而重复使用了数次。  


#### 流的操作 {#major8}

当把一个数据结构包装成 Stream 后，就要开始对里面的元素进行各类操作了。常见的操作可以归类如下。

* 中间操作（Intermediate）：

`map` \(`mapToInt`, `flatMap` 等\)、 `filter`、 `distinct`、 `sorted`、 `peek`、 `limit`、 `skip`、 `parallel`、 `sequential`、 `unordered`

* 终止操作（Terminal）：

`forEach`、 `forEachOrdered`、 `toArray`、 `reduce`、 `collect`、 `min`、 `max`、 `count`、 `anyMatch`、 `allMatch`、 `noneMatch`、 `findFirst`、 `findAny`、 `iterator`

* Short-circuiting：

`anyMatch`、 `allMatch`、 `noneMatch`、 `findFirst`、 `findAny`、 `limit`

我们下面看一下 Stream 的比较典型用法。

1、**map/flatMap**

map：把 input Stream 的每一个元素，映射成 output Stream 的另外一个元素。

flatMap：一对多映射，把 input Stream 中的层级结构扁平化，就是将最底层元素抽出来放到一起。

转换为大写：把所有的单词转换为大写

```java
List<String> output = wordList.stream()
        .map(String::toUpperCase)
        .collect(Collectors.toList());
```

 平方数：生成一个整数 list 的平方数 {1, 4, 9, 16}

```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4);
List<Integer> squareNums = nums.stream()
        .map(n -> n * n)
        .collect(Collectors.toList());
```

 一对多

```text
Stream<List<Integer>> inputStream = Stream.of(
        Arrays.asList(1),
        Arrays.asList(2, 3),
        Arrays.asList(4, 5, 6)
);
Stream<Integer> outputStream = inputStream.
        flatMap((childList) -> childList.stream());
```

 **2、filter**

filter 对原始 Stream 进行某项测试，通过测试的元素被留下来生成一个新 Stream。（即过滤）。

留下偶数

```java
Integer[] sixNums = {1, 2, 3, 4, 5, 6};
Integer[] evens = Stream.of(sixNums)
        .filter(n -> n % 2 == 0)
        .toArray(Integer[]::new);
```

 把单词挑出来：首先把每行的单词用 flatMap 整理到新的 Stream，然后保留长度不为 0 的，就是整篇文章中的全部单词了。

```java
List<String> output = reader.lines()
        .flatMap(line -> Stream.of(line.split(REGEXP)))
        .filter(word -> word.length() > 0)
        .collect(Collectors.toList());
```

 **3、forEach**

forEach 方法接收一个 Lambda 表达式，然后在 Stream 的每一个元素上执行该表达式。

打印姓名：对一个人员集合遍历，找出男性并打印姓名

```java
// Java 8
roster.stream()
        .filter(p -> p.getGender() == Person.Sex.MALE)
        .forEach(p -> System.out.println(p.getName()));
// Pre-Java 8
for (Person p : roster) {
    if (p.getGender() == Person.Sex.MALE) {
        System.out.println(p.getName());
    }
}
```

 可以看出来，forEach 是为 Lambda 而设计的，保持了最紧凑的风格。而且 Lambda 表达式本身是可以重用的，非常方便。当需要为多核系统优化时，可以 parallelStream\(\).forEach\(\)，只是此时原有元素的次序没法保证，并行的情况下将改变串行时操作的行为，此时 forEach 本身的实现不需要调整，而 Java8 以前的 for 循环 code 可能需要加入额外的多线程逻辑。

但一般认为，forEach 和常规 for 循环的差异不涉及到性能，它们仅仅是函数式风格与传统 Java 风格的差别。

另外一点需要注意，forEach 是 terminal 操作，因此它执行后，Stream 的元素就被“消费”掉了，你无法对一个 Stream 进行两次 terminal 运算。

相反，具有相似功能的 intermediate 操作 peek 可以达到上述目的。

peek 对每个元素执行操作并返回一个新的 Stream

```java
Stream.of("one", "two", "three", "four")
        .filter(e -> e.length() > 3)
        .peek(e -> System.out.println("Filtered value: " + e))
        .map(String::toUpperCase)
        .peek(e -> System.out.println("Mapped value: " + e))
        .collect(Collectors.toList());
```

注意：forEach 不能修改自己包含的本地变量值，也不能用 break/return 之类的关键字提前结束循环。

4、**findFirst**

这是一个 termimal 兼 short-circuiting 操作，它总是返回 Stream 的第一个元素，或者空。

这里比较重点的是它的返回值类型：Optional。这也是一个模仿 Scala 语言中的概念，作为一个容器，它可能含有某值，或者不包含。使用它的目的是尽可能避免 `NullPointerException`。

Optional 的两个用例

```java
public static void print(String text) {
    // Java 8
    Optional.ofNullable(text).ifPresent(System.out::println);
    // Pre-Java 8
    if (text != null) {
        System.out.println(text);
    }
}

public static int getLength(String text) {
    // Java 8
    return Optional.ofNullable(text).map(String::length).orElse(-1);
    // Pre-Java 8
    return if (text != null) ? text.length() : -1;
}
```

 在更复杂的 if \(xx != null\) 的情况中，使用 Optional 代码的可读性更好，而且它提供的是编译时检查，能极大的降低 NPE 这种 Runtime Exception 对程序的影响，或者迫使程序员更早的在编码阶段处理空值问题，而不是留到运行时再发现和调试。

Stream 中的 findAny、max/min、reduce 等方法等返回 Optional 值。还有例如 IntStream.average\(\) 返回 OptionalDouble 等等。

**5、reduce**

主要作用是把 Stream 元素组合起来。它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面 Stream 的第一个、第二个、第 n 个元素组合。从这个意义上说，字符串拼接、数值的 sum、min、max、average 都是特殊的 reduce。例如 Stream 的 sum 就相当于

```text
Integer sum = integers.reduce(0, (a, b) -> a+b); 
//或者
Integer sum = integers.reduce(0, Integer::sum);
```

也有没有起始值的情况，这时会把 Stream 的前面两个元素组合起来，返回的是 Optional。

reduce 的用例

```java
// 字符串连接，concat = "ABCD"
String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat);
// 求最小值，minValue = -3.0
double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min);
// 求和，sumValue = 10, 有起始值
int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
// 求和，sumValue = 10, 无起始值
sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
// 过滤，字符串连接，concat = "ace"
concat = Stream.of("a", "B", "c", "D", "e", "F")
        .filter(x -> x.compareTo("Z") > 0)
        .reduce("", String::concat);
```

 上面代码，例如第一个示例的 reduce\(\)，第一个参数（空白字符）即为起始值，第二个参数（String::concat）为 BinaryOperator。这类有起始值的 reduce\(\) 都返回具体的对象。而对于第四个示例没有起始值的 reduce\(\)，由于可能没有足够的元素，返回的是 Optional，请留意这个区别。

**6、limit/skip**

limit 返回 Stream 的前面 n 个元素；

skip 则是扔掉前 n 个元素（它是由一个叫 subStream 的方法改名而来）。

limit 和 skip 对运行次数的影响

```java
public void testLimitAndSkip() {
    List<Person> persons = new ArrayList();
    for (int i = 1; i <= 10000; i++) {
        Person person = new Person(i, "name" + i);
        persons.add(person);
    }
    
    List<String> personList2 = persons.stream()
            .map(Person::getName)
            .limit(10)
            .skip(3)
            .collect(Collectors.toList());
    System.out.println(personList2);
}
private class Person {
    
    public int no;
    private String name;
    public Person (int no, String name) {
        this.no = no;
        this.name = name;
    }
    public String getName() {
        System.out.println(name);
        return name;
    }
}
//输出结果如下
name1
name2
name3
name4
name5
name6
name7
name8
name9
name10
[name4, name5, name6, name7, name8, name9, name10]
```

 这是一个有 10，000 个元素的 Stream，但在 short-circuiting 操作 limit 和 skip 的作用下，管道中 map 操作指定的 getName\(\) 方法的执行次数为 limit 所限定的 10 次，而最终返回结果在跳过前 3 个元素后只有后面 7 个返回。

有一种情况是 limit/skip 无法达到 short-circuiting 目的的，就是把它们放在 Stream 的排序操作后，原因跟 sorted 这个 intermediate 操作有关：此时系统并不知道 Stream 排序后的次序如何，所以 sorted 中的操作看上去就像完全没有被 limit 或者 skip 一样。

 limit 和 skip 对 sorted 后的运行次数无影响：对上面的代码做了微调，首先对 5 个元素的 Stream 排序，然后进行 limit 操作。

```java
public void testLimitAndSkip() {
    List<Person> persons = new ArrayList();
    for (int i = 1; i <= 5; i++) {
        Person person = new Person(i, "name" + i);
        persons.add(person);
    }
    List<Person> personList2 = persons.stream()
            .sorted((p1, p2) -> p1.getName().compareTo(p2.getName()))
            .limit(2)
            .collect(Collectors.toList());
    System.out.println(personList2);
}
//输出如下
name2
name1
name3
name2
name4
name3
name5
name4
[stream.StreamDW$Person@816f27d, stream.StreamDW$Person@87aac27]
```

即虽然最后的返回元素数量是 2，但整个管道中的 sorted 表达式执行次数没有像前面例子相应减少。 

最后有一点需要注意的是，对一个 parallel 的 Steam 管道来说，如果其元素是有序的，那么 limit 操作的成本会比较大，因为它的返回对象必须是前 n 个也有一样次序的元素。取而代之的策略是取消元素间的次序，或者不要用 parallel Stream。

** 7、sorted**

对 Stream 的排序通过 sorted 进行，它比数组的排序更强之处在于你可以首先对 Stream 进行各类 map、filter、limit、skip 甚至 distinct 来减少元素数量后，再排序，这能帮助程序明显缩短执行时间。

优化上面先sorted后limit的例子：改为排序前进行 limit 和 skip

```java
List<Person> persons = new ArrayList();
for (int i = 1; i <= 5; i++) {
    Person person = new Person(i, "name" + i);
    persons.add(person);
}
List<Person> personList2 = persons.stream()
        .limit(2)
        .sorted((p1, p2) -> p1.getName().compareTo(p2.getName()))
        .collect(Collectors.toList());
System.out.println(personList2);
//输出如下
name2
name1
[stream.StreamDW$Person@6ce253f1, stream.StreamDW$Person@53d8d10a]
```

 当然，这种优化是有 business logic 上的局限性的：即不要求排序后再取值。

**8、min/max/distinct**

min 和 max 的功能也可以通过对 Stream 元素先排序，再 findFirst 来实现，但前者的性能会更好，为 O\(n\)，而 sorted 的成本是 O\(n log n\)。同时它们作为特殊的 reduce 方法被独立出来也是因为求最大最小值是很常见的操作。

找出最长一行的长度

```java
BufferedReader br = new BufferedReader(new FileReader("c:\\SUService.log"));
int longest = br.lines()
        .mapToInt(String::length)
        .max()
        .getAsInt();
br.close();
System.out.println(longest);
```

 使用 distinct 来找出不重复的单词：找出全文的单词，转小写，并排序

```java
List<String> words = br.lines()
        .flatMap(line -> Stream.of(line.split(" ")))
        .filter(word -> word.length() > 0)
        .map(String::toLowerCase)
        .distinct()
        .sorted()
        .collect(Collectors.toList());
br.close();
System.out.println(words);
```

 **9、Match**

Stream 有三个 match 方法，从语义上说：

* allMatch：Stream 中全部元素符合传入的 predicate，返回 true
* anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true
* noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true

它们都不是要遍历全部元素才能返回结果。例如 allMatch 只要一个元素不满足条件，就 skip 剩下的所有元素，返回 false。对清单 13 中的 Person 类稍做修改，加入一个 age 属性和 getAge 方法。

使用 Match

```java
boolean isAllAdult = persons.stream()
        .allMatch(p -> p.getAge() > 18);
System.out.println("All are adult? " + isAllAdult);

boolean isThereAnyChild = persons.stream()
        .anyMatch(p -> p.getAge() < 12);
System.out.println("Any child? " + isThereAnyChild);
```

 



 

## 参考

[五分钟学习Java8的流编程](https://juejin.im/post/5b07f4536fb9a07ac90da4e5?utm_source=gold_browser_extension)

[Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/)：内容来源

