# fast-fail

相信不少开发者在使用集合时，都遇到过`ConcurrentModificationException`这个异常。在JDK的Collection和Map中的各个实现类，尤其是内部的迭代器Iterator实现中，我们经常会看到类似于这样的代码注释：

ArrayList

> 注意，迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出 ConcurrentModificationException。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误的做法：迭代器的快速失败行为应该仅用于检测 bug。

HashMap

> 注意，迭代器的快速失败行为不能得到保证，一般来说，存在非同步的并发修改时，不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。

fail-fast，即快速失败，是Java集合的一种**错误检测机制：**当多个线程对集合进行结构上的改变的操作时，程序会尽可能地抛出异常（通常为ConcurrentModificationException）。

规则为：

* 抛出ConcurrentModificationException异常，程序一定出现了非法操作，如
  * **使用多线程去读写ArraList等线程不安全的集合**
  * **单线程循环操作时在不恰当的位置remove了元素（进行了结构的改变）**
* 不抛出ConcurrentModificationException异常，并不表示程序没有错误，即非法操作可能并不会抛出该类异常。
* ConcurrentModificationException异常的抛出，

**即fail-fast只是一种bug检测或错误检测的机制，不能保证程序的正确性。**

### ConcurrentModificationException异常抛出场景

1、**多线程去读写ArrayList等线程不安全的集合**

```java
@Test
public void exception1() throws InterruptedException {
    List<Integer> list = new ArrayList<>();
    ExecutorService threadPool = Executors.newFixedThreadPool(3);
    for (int i = 0; i < 1000; i++) {
        final int a = i;
        threadPool.execute(() -> {
            list.add(a);
        });
    }
    Collections.sort(list);
    TimeUnit.SECONDS.sleep(2);
}
//抛出如下异常（省略部分异常信息）：
java.util.ConcurrentModificationException
	at java.util.ArrayList.sort(ArrayList.java:1456)
	at java.util.Collections.sort(Collections.java:141)
	at com.maxwell.learning.common.ListExample.exception1(ListExample.java:123)
	at java.util.ArrayList.forEach(ArrayList.java:1249)
```

2、**单线程循环操作时，使用在不恰当方式修改Collection结构**

```java
@Test
public void exception2() throws InterruptedException {
    List<Integer> list = new ArrayList<>(Arrays.asList(1, 3, 5, 7));
    Iterator<Integer> it = list.iterator();
    int index = 0;
    while (it.hasNext()) {
        System.out.println(it.next());
        index++;
        if (index == 2) {
            //将此处换为：list.clear()或者list.add()等操作，同样会快速失败
            list.remove(index);
        }
    }
}

//抛出如下异常（省略部分异常信息）：
java.util.ConcurrentModificationException
	at java.util.ArrayList.sort(ArrayList.java:1456)
	at java.util.Collections.sort(Collections.java:141)
	at com.maxwell.learning.common.ListExample.exception2(ListExample.java:133)
	at java.util.ArrayList.forEach(ArrayList.java:1249)
```

### 如何避免

一旦抛出诸如ConcurrentModificationException的异常，那么一定是对Collection或Map的使用方式不当引发的，比如在将线程非安全的集合用在了多线程的场景下，或者在Iterator的迭代中，调用list的修改操作对集合进行了结构修改。在开发中，避免此类使用不当的情况，多了解各个集合类的适用场景和正确用法，是解决问题的根本。







