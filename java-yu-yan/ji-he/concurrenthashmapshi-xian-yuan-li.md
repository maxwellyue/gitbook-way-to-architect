# 数据结构

ConcurrentHashMap的底层数据结构是**数组+单向链表**。与HashMap不同，它的数组元素的数据结构为Segment，Segment本身又是数组+链表的结构，如下所示：



HashTable容器在竞争激烈的并发环境下表现出效率低下的原因是所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的**锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。**

# 初始化

ConcurrentHashMap初始化方法是通过`initialCapacity/loadFacto/concurrencyLevel`几个参数来初始化segments数组，每个segment里的HashEntry数组 。

**初始化segments数组**

```java
if (concurrencyLevel > MAX_SEGMENTS)
    concurrencyLevel = MAX_SEGMENTS;

// 找到最小的大于等于concurrencyLevel的值，作为Segment数组的长度
int sshift = 0;
int ssize = 1;//segments数组的长度
while (ssize < concurrencyLevel) {
    ++sshift;
    ssize <<= 1;
}
segmentShift = 32 - sshift;
segmentMask = ssize - 1;
this.segments = Segment.newArray(ssize);
```

初始化segmentShift和segmentMask。这两个全局变量在定位segment时的哈希算法里需要使用。

**初始化每个Segment**

initialCapacity是ConcurrentHashMap的初始化容量，loadfactor是每个segment的负载因子，在构造方法里需要通过这两个参数来初始化数组中的每个segment。

```java
if (initialCapacity > MAXIMUM_CAPACITY)
    initialCapacity = MAXIMUM_CAPACITY;
int c = initialCapacity / ssize;
if (c * ssize < initialCapacity)
    ++c;
int cap = 1;//每个segment里HashEntry数组的长度
while< (cap < c)
    cap <<= 1;
for (int i = 0; i < this.segments.length; ++i)
    this.segments[i] = new Segment<K,V>(cap, loadFactor);
```

segment里HashEntry数组的长度=initialCapacity除以ssize的倍数c，如果c大于1，就会取大于等于c的2的N次方值，所以cap不是1，就是2的m次幂。segment的容量threshold＝\(int\)cap\*loadFactor，默认情况下initialCapacity等于16，loadfactor等于0.75，通过运算cap等于1，threshold等于零。

# 定位Segment

**计算key的哈希值**

在插入和获取元素的时候，必须先通过哈希算法定位到Segment。可以看到ConcurrentHashMap会首先使用Wang/Jenkins hash的变种算法对元素的hashCode进行一次再哈希。

```java
private int hash(Object k) {
    int h = hashSeed;

    if ((0 != h) && (k instanceof String)) {
        return sun.misc.Hashing.stringHash32((String) k);
    }
    
    //重点是以下内容
    h ^= k.hashCode();
    // Spread bits to regularize both segment and index locations,
    // using variant of single-word Wang/Jenkins hash.
    h += (h <<  15) ^ 0xffffcd7d;
    h ^= (h >>> 10);
    h += (h <<   3);
    h ^= (h >>>  6);
    h += (h <<   2) + (h << 14);
    return h ^ (h >>> 16);
}
```

Wang/Jenkins hash：通过再哈希能让数字的每一位都能参加到哈希运算当中，从而减少哈希冲突。

**定位segment：TODO**

```java
private Segment<K,V> segmentForHash(int h) {
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    return (Segment<K,V>) UNSAFE.getObjectVolatile(segments, u);
}
```

# get

先通过哈希值定位到segment，再通过哈希算法定位到元素：

```java
public V get(Object key) {
    int hash = hash(key.hashCode());
    return segmentFor(hash).get(key, hash);
}
```

get操作的高效之处在于**整个get过程不需要加锁，除非读到的值是空的才会加锁重读**，我们知道HashTable容器的get方法是需要加锁的，那么ConcurrentHashMap的get操作是如何做到不加锁的呢？原因是它的**get方法里将要使用的共享变量都定义成volatile**（如用于统计当前Segement大小的count字段和用于存储值的HashEntry的value）。在get操作里只需要读不需要写共享变量count和value，所以可以不用加锁。之所以不会读到过期的值，是根据java内存模型的happen before原则，对volatile字段的写入操作先于读操作，即使两个线程同时修改和获取volatile变量，get操作也能拿到最新的值，这是用volatile替换锁的经典应用场景。

# Put

由于put方法里需要对共享变量进行写入操作，所以为了线程安全，在操作共享变量时必须得加锁。Put方法首先定位到Segment，然后在Segment里进行插入操作。插入操作需要经历两个步骤，第一步判断是否需要对Segment里的HashEntry数组进行扩容，第二步定位添加元素的位置然后放在HashEntry数组里。

**判断是否需要扩容**：在插入元素前会先判断Segment里的HashEntry数组是否==容量（threshold），如果超过阀值，数组进行扩容。Segment的扩容判断比HashMap更恰当：HashMap是在插入元素后判断元素是否==容量，如果到达了就进行扩容，但是很有可能扩容之后再没有新元素插入，这时HashMap就进行了一次无效的扩容。

**如何扩容**：扩容的时候首先会创建一个两倍于原容量的数组，然后将原数组里的元素进行再hash后插入到新的数组里。为了高效，ConcurrentHashMap**不会对整个容器进行扩容，而只对某个segment进行扩容**。

## size

如果我们要统计整个ConcurrentHashMap里元素的大小，就必须统计所有Segment里元素的大小后求和。Segment里的全局变量count是一个volatile变量，那么在多线程场景下，我们是不是直接把所有Segment的count相加就可以得到整个ConcurrentHashMap大小了呢？不是的，虽然相加时可以获取每个Segment的count的最新值，但是拿到之后可能累加前使用的count发生了变化，那么统计结果就不准了。所以最安全的做法，是在统计size的时候把所有Segment的put，remove和clean方法全部锁住，但是这种做法显然非常低效。 因为在累加count操作过程中，之前累加过的count发生变化的几率非常小，所以**ConcurrentHashMap的做法是先尝试2次通过不锁住Segment的方式来统计各个Segment大小，如果统计的过程中，容器的count发生了变化，则再采用加锁的方式来统计所有Segment的大小**。

那么ConcurrentHashMap是如何判断在统计的时候容器是否发生了变化呢？**使用modCount变量**，在put , remove和clean方法里操作元素前都会将变量modCount进行加1，那么在统计size前后比较modCount是否发生变化，从而得知容器的大小是否发生变化。



内容来源：[聊聊并发（四）——深入分析ConcurrentHashMap](http://www.infoq.com/cn/articles/ConcurrentHashMap)

  
  


#  

  


