## 数据结构 {#数据结构-1}

Java 7为实现并行访问，引入了Segment这一结构，实现了分段锁，理论上最大并发度与Segment个数相等。Java 8为进一步提高并发性，摒弃了分段锁的方案，而是直接使用一个大的数组。同时为了提高哈希碰撞下的寻址性能，Java 8在链表长度超过一定阈值（8）时将链表转换为红黑树。

即JDK1.8中，ConcurrentHashMap的数据结构与1.8中的HashMap结构类似，都是：**数组+单向链表/红黑树**。

## Hash算法 {#寻址方式-1}

Java 8的ConcurrentHashMap同样是通过Key的哈希值与数组长度取模确定该Key在数组中的索引。同样为了避免不太好的Key的hashCode设计，它通过如下方法计算得到Key的最终哈希值。不同的是，Java 8的ConcurrentHashMap作者认为引入红黑树后，即使哈希冲突比较严重，寻址效率也足够高，所以作者并未在哈希值的计算上做过多设计，只是将Key的hashCode值与其高16位作异或并保证最高位为0（从而保证最终结果为正整数）。

```java
//计算key的哈希值
h = key.hashCode();
//ConcurrentHashMap中的hash算法，其中HASH_BITS为0x7fffffff，即Integer.MAX_VALUE的值
h = (h ^ (h >>> 16)) & HASH_BITS
//HashMap中的hash算法
h = h ^ (h >>> 16)

//计算table数组位置
(n - 1) & h
```

相比HashMap，多了一步与运算：保证最高位为0，从而保证最终结果为正整数。

## put {#同步方式-1}

**大体思路**：

* 如果`table`数组还没有初始化，则使用`CAS`进行初始化
* 如果`table`数组中`i`位置处元素为空，则使用`CAS`将`table[i]`的值设置为value
* 如果其他线程正在对`table`数组进行扩容，则当前线程去协助其进行扩容
* 其他情况，则使用`synchronized`锁住`table[i]`这个元素（链表表头或红黑树根节点），并将元素追加插入到链表或红黑树中；插入后，检查是否需要将该桶的数据结构由链表转化为红黑树。

* 成功设置`<key, value>`后，检查是否需要进行扩容

> 要想对链表或红黑树进行put操作，必须拿到表头或根节点，所以，锁住了表头或根节点，就相当于锁住了整个链表或者整棵树。这个设计与分段锁的思想一致，只是其他读取操作需要用cas来保证。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        //无限循环，如果其他线程正在修改table数组，则当前循环可能失败，继续下次尝试，直到成功
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            //如果table数组还没有初始化，则使用CAS进行初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            //如果table数组中i位置处元素为空，则使用CAS将table[i]的值设置为value
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            //如果其他线程正在对table数组进行扩容，则当前线程去协助其进行扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            //其他情况，则锁住table[i]这个元素（链表表头或红黑树根节点），并将元素追加插入到链表或红黑树中。
            else {
                V oldVal = null;
                //锁住表头或根节点
                synchronized (f) {
                    //桶中为链表的情况
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        //桶中为红黑树的情况
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                //检查当前桶的结构是否需要由链表转换为红黑树
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        //检查是否需要扩容，内部也使用了CAS操作进行
        addCount(1L, binCount);
        return null;
    }
```

**get**

大体思路：

* 根据hash值确定节点所在的table数组的位置i
* 如果table\[i\]恰好就是要找的key，直接返回table\[i\]
* 如果table的i处为链表结构，查从链表中查找该键值对并返回
* 如果table的i处为红黑树结构或table\[i\]为已迁移节点（发生了扩容），则调用对应的方法去查找并返回
* 没有找到，返回null

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    //计算key的hash值
    int h = spread(key.hashCode());
    //根据hash值确定节点所在的table数组的位置i
    if ((tab = table) != null && (n = tab.length) > 0 &&
        //定位桶的位置
        (e = tabAt(tab, (n - 1) & h)) != null) {
        //如果table[i]的key与传入的key相同（key的哈希值相同， key是equals的），则直接返回table[i]
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //如果table[i]为特殊节点（红黑树或者ForwardingNode等），则去其中其查找
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        //如果table的i处为链表，查从链表中查找该键值对并返回
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    //没有找到
    return null;
}
```

整个get操作都是无锁的：（1）table数组被volatile关键字修饰；（2）元素的数据结构均为Node，其key值和hash值都由final修饰，不可变更，其value及对下一个元素的引用由volatile修饰，可见性也有保障。

```java
static class Node<K,V> implements Map.Entry<K,V> {
  final int hash;
  final K key;
  volatile V val;
  volatile Node<K,V> next;
}
```

对于key对应的数组元素的可见性，由Unsafe的getObjectVolatile方法保证。

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
  return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
```

## 总结

x相比







## 内容来源

[Java进阶（六）从ConcurrentHashMap的演进看Java多线程核心技术](http://www.jasongj.com/java/concurrenthashmap/)

[探索jdk8之ConcurrentHashMap 的实现机制](https://www.cnblogs.com/huaizuo/p/5413069.html)

[Java8集合源码解析——ConcurrentHashMap](http://www.voidcn.com/article/p-kuonwbvr-bqt.html)

[ConcurrentHashMap源码分析（JDK8） get/put/remove方法分析](https://www.jianshu.com/p/5bc70d9e5410)

  


