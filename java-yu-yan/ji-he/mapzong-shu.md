**Map**  
Map没有继承Collection接口，Map提供key到value的映射。  
一个Map中不能包含相同的key，每个key只能映射一个value。Map接口提供3种集合的视图，Map的内容可以被当作一组key集合，一组value集合，或者一组key-value映射。![](/assets/map.png)**AbstractMap**  
实现了Map中的绝大部分函数接口。它减少了“Map的实现类”的重复编码。

**SortedMap**  
实现该接口，说明该Map依据key进行排序。

```
A Map that further provides a total ordering on its keys
```

**NavigableMap**

实现该接口，说明Map具有更多定位（或搜索）元素的方法。

```
A SortedMap extended with navigation methods returning the closest matches for given search targets。
All of these methods are designed for locating, not traversing entries.
```

**Dictionary**

```
This class is obsolete.  
New implementations should implement the Map interface, rather than extending this class
```

# java.util

**HashMap：**数据结构为链表数组，key和value都允许null，未实现线程同步

> （1）HashMap默认的容量大小是16；增加容量时，每次将容量变为原始容量x2。  
> （2）HashMap添加元素时，是使用自定义的哈希算法。  
> （3）通过Iterator迭代器遍历时：“从前向后”的遍历数组；再对数组具体某一项对应的链表，从表头开始进行遍历。

**Hashtable：**数据结构为链表数组，key和value都不允许null，实现了线程同步

> （1）Hashtable默认的容量大小是11；增加容量时，每次将容量变为“原始容量x2 + 1”。  
> （2）Hashtable没有自定义哈希算法，而直接采用的key的hashCode\(\)。  
> （3）通过Iterator迭代器遍历时：“从后向前”的遍历数组；再对数组具体某一项对应的链表，从表头开始进行遍历。

**WeakHashMap**

> WeakHashMap是一种改进的HashMap，它**对key实行弱引用**，如果一个key不再被外部所引用，那么该key可以被GC回收。

**LinkedHashMap**

> LinkedHashMap 是HashMap的一个子类，保存了记录的插入顺序。  
> 在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的。  
> 也可以在构造时用带参数，按照应用次数排序。  
> 在遍历的时候会比HashMap慢，不过有种情况例外，当HashMap容量很大，实际数据较少时，遍历起来可能会比 LinkedHashMap慢，因为LinkedHashMap的遍历速度只和实际数据有关，和容量无关，而HashMap的遍历速度和他的容量有关。  
> **区别在于HashMap并不是按插入次序顺序存放的，而LinkedHashMap是顺序存放的。  
> 如果需要输出的顺序和输入的相同,那么用LinkedHashMap 可以实现,它还可以按读取顺序来排列。**

**TreeMap**

> TreeMap实现SortMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator 遍历TreeMap时，得到的记录是排过序的。  
> **如果要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。**

## java.util.concurrent

**ConcurrentHashMap**

HashMap的线程安全版本

**ConcurrentSkipListMap**

基于跳表实现的支持排序的并发的Map

## 总结

| 类型 | 使用场景 |
| :--- | :--- |
| HashMap | 最常用 |
| LinkedHashMap | 遍历key的顺序，与put的顺序一致的时候 |
| TreeMap | 按key进行排序的时候 |
| ConcurrentHashMap | 并发，即多线程环境使用 |
| ConcurrentSkipListMap | 按key进行排序，且要并发，即在多线程环境使用 |

 如果仅仅需要一个高效的键值对容器，则HashMap是最好的选择；

如果想要插入的键值对保持插入时的顺序，则LinkedHashMap是最好的选择；

如果不仅要存储键值对，还要键值对按照key来进行排序，则TreeMap是最好的选择；

如果在多线程环境下，ConcurrentHashMap是最好的选择；

如果在多线程环境下，还要对键值对按照key来进行排序，则ConcurrentSkipListMap是最好的选择；

遗憾的是，JDK并未提供LinkedHashMap的线程安全版本，如果想在多线程环境下使用LinkedHashMap，则只能使用synchronizedMap进行包装：

```java
Map<K,V> map = Collections.synchronizedMap(new LinkedHashMap<>());
```

那为什么没有TreeMap的线程安全版本？TreeMap的目的是实现key的排序，其底层数据结构为红黑树，如果想要实现基于红黑树的并发容器，由于红黑树的平衡操作需要锁住整个树，其性能势必较差，而ConcurrentSkipListMap是基于跳表实现的并发键值对容器，同样支持排序，但由于跳表的结构可以实现分段加锁，即相比红黑树，其锁粒度可以更小，性能会更高。













