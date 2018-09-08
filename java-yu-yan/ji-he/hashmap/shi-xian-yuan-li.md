## 概述

HashMap是Map接口基于哈希表\(hash table\)的实现。

在HashMap内部，哈希表的实现是通过数组+单链表/红黑树（JDK1.8）来完成的，示意图如下：![](/assets/hashmap.png)

**大体的思路**：HashMap的内部维护一个数组（Node&lt;K,V&gt;\[\] table），该数组的每一个位置存放一个bucket（桶或哈希桶），当添加键值对的时候，首先确定要将该键值对放在哪个桶里（table数组的哪个位置），该桶的结构一开始是单链表，但是如果在某一个桶中放入了过多的键值对，那么这个桶的结构就会由单链表转换为红黑树。

这里说的table数组每个位置存放一个bucket的意思是：实际上table数组的每个位置真正存放的就是一个结点Node，只不过该位置处要存放不止一个Node，这些Node就会形成链表或者红黑树，桶则是指链表或红黑树。我觉得不必过于纠结这些概念，只要明白桶就是装东西的，水桶装水，HashMap里的桶装键值对。

这里的桶只是抽象上的概念，简单来讲，HashMap首先维护一个键值对数组，不在这个数组里的键值对通过与数组中某个位置的键值对发生某种关系，形成单链表或者红黑树结构。HashMap的一切操作都在维护着这种关系，以便达到高效的查找和增删效率。

在源码的注释中，经常出现的bin就是指的键值对的封装体：TreeNode&lt;K,V&gt;或者Node&lt;K,V&gt;。  
在源码的注释中，经常出现的bucket是指桶（或叫做哈希桶）：盛放了一组结点的单链表或红黑树。

下图虚线内的Node就是table数组实际存放的键值对：

![](/assets/table数组实际存放的键值对.png)下图示意了一个单链表和一颗红黑树的范畴  
![](/assets/单链表和红黑树示意图.png)

### 键值对的载体：Node&lt;K,V&gt;和TreeNode&lt;K,V&gt;

对于HashMap内部，任意一个键值对都是以Node&lt;K,V&gt;的形式封装的。

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;//哈希值
    final K key;//键
    V value;//值
    Node<K,V> next;//下一个结点
    ...
}
```

可以看到，Node中包含了属性`Node<K,V> next`，也就是下一个结点。HashMap中的单链表结构就是通过这种方式实现的。

TreeNode&lt;K,V&gt;是Node&lt;K,V&gt;的子类（扩展类），实现了红黑树的性质

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    ...
}

//下面这段是LinkedHashMap的代码，LinkedHashMap是HashMap的子类。
static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
}
```

### 主要成员变量

```java
//默认初始容量（即HashMap可以存放的键值对的个数）：16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

//HashMap所能存放键值对的最大数量。
static final int MAXIMUM_CAPACITY = 1 << 30;

//如果构造方法中未指明装载因子，则使用此默认值
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//当桶里的Node（可以理解为键值对）数大于该值时，该桶的结构就会由链表转变为红黑树。
//该值必须大于2，且最小为8，以此保证可以在桶内键值对减小时桶结构由红黑树退化为链表。
static final int TREEIFY_THRESHOLD = 8;

//当桶内的Node（可以理解为键值对）数量小于该值时，桶结构就会由红黑树转变为链表。
//该值最大为6 。
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * The smallest table capacity for which bins may be treeified.
 * (Otherwise the table is resized if too many nodes in a bin.)
 * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
 * between resizing and treeification thresholds.
 */
//桶结构开始树化时，数组的最小长度，否则如果很多Node在一个桶中，数组就会resized，
//将元素更均匀地分布在各个位置
static final int MIN_TREEIFY_CAPACITY = 64;


//Node数组，这个就是HashMap中数据结构为数组+链表+红黑树中的数组
//数组的长度永远是2的倍数
transient Node<K,V>[] table;

/**
 * Holds cached entrySet(). Note that AbstractMap fields are used
 * for keySet() and values().
 * 待研究
 */
transient Set<Map.Entry<K,V>> entrySet;

//HashMap中键值对的个数
transient int size;

//结构修改次数（结构修改是指改变了HashMap中键值对的个数或者内部的重哈希。
//为了fail-fast机制用，抛出ConcurrentModificationException）
transient int modCount;

/**
 * The next size value at which to resize (capacity * load factor).
 *
 * @serial
 */
// (The javadoc description is true upon serialization.
// Additionally, if the table array has not been allocated, this
// field holds the initial array capacity, or zero signifying
// DEFAULT_INITIAL_CAPACITY.)
// 发生resize的阈值（键值对个数超过该值，HashMap内部就会进行resize操作）
// 如果数组还没有分配，该值为数组的初始容量
int threshold;

//哈希表的装载因子：也就是装满程度
final float loadFactor;
```

### put

```java
//存放键值对，通过调用putVal方法完成
//如果之前通过该key放过value，该方法会用value替换之前的value
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

**putVal\(\)的大体思路**：先确定该键值对应该存放在哪个桶中，即确定放在table\[i\]中的i值；如果该桶为空，即table的i位置处还没有存放键值对，则直接将键值对放在table数组中；如果该桶不为空，并且桶结构为红黑树，则将键值对插入到该红黑树中；如果该桶不为空，并且桶结构为链表，则将该键值对插入到链表尾端（插入后，如果链表长度大于8，则将该链表结构转换为红黑树结构）。最后，检查是否需要扩容。

```java
/**
 * 该方法是Map接口中put相关方法的真正实现。
 * @param hash 键key的哈希值
 * @param key 键值对中的键
 * @param value 键值对中的值
 * @param onlyIfAbsent 见名只意：如果为true，那么就不会替换掉旧value
 *         （如果之前通过该key放过value）。
 * @param evict if false, the table is in creation mode.
 * @return 该键对应的之前的值（如果之前该键存放过值，没有的话返回null）
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab;
    Node<K,V> p;
    int n, i;
    //如果HashMap的数组table未初始化或者table的长度为0，则对table进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;

    //先用(n - 1) & hash 确定该键值对应该存放在哪个桶中
    //（table的哪个位置处：table[i]），
    //如果桶为空，则新生成结点（Node，非TreeNode）放入桶中
    //(此时，由于桶中只有一个结点，这个结点是放在数组table中)
    if ((p = tab[i = (n - 1) & hash]) == null)//这里，p=table的i位置处的桶中的第一个元素
        tab[i] = newNode(hash, key, value, null);
    else {
        //如果该键值对要放的位置已经存在非空桶,也就是要放在桶里的某个位置：
        //有两种情况：该桶结构为链表或该桶结构为红黑树
        //如果为链表，则又有两种情况，一种是在加入该键值对后该桶依然保持链表结构，
        //第二种就是加入该键值对后该桶结构转换为红黑树
        Node<K,V> e;//这里的e记录的就是要存放的新的键值对
        K k;//这里的k就是要存放的新的键值对的键
        //上面已经对p赋值过了：p=table的i位置处的桶中的第一个元素，这个i就是该要存放的键值对应该放的位置桶的位置
        //如果要新存放的键值对中的键与p结点的键相同()
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //如果p是红黑树中的结点的类型，则将结点放入红黑树中
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {//这种情况是：新的键值对要放的桶为链表结构，把该键值对放到链表中
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {//循环达到链表的尾部，注意这里的赋值，e=p.next
                    //在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    //如果插入新结点后，链表中结点数量超过了阈值TREEIFY_THRESHOLD，则将该链表结构转换为红黑树结构
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    //跳出循环
                    break;
                }
                //在每一次循环中，判断当前链表中的结点的key是否与要存放的新的键值对的key相同
                //相同就跳出循环（这里的e已经在之前被赋值为p.next）
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;

                //这一句是用于遍历桶中的链表，与前面的e = p.next组合，其他没啥用
                p = e;
            }
        }
        //表示在桶中找到key值、hash值与插入元素相等的结点（变量e就指向那个结点）
        //此时，只要更新e的值就可以了。
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //此时新键值对已经添加到了HashMap中，如果此时HashMap中的键值对个数大于了阈值，就进行扩容resize
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### resize

resize\(\)方法有两个作用：初始化table数组；将table数组长度翻倍。

在将table数组长度翻倍的时候，还要重新计算每个键值对应该在的位置，并重新生成链表或者红黑树，这是非常消耗性能的。所以，在使用HashMap的时候，如果能预估出元素的最大容量，最好指定这个容量值。

**扩容时机**：
当put时，如果发现目前的bucket占用程度已经超过了Load Factor所希望的比例，那么就会发生resize。
**大体过程**：
先将table数组长度变为原来的2倍，然后将table中的键值对重新分配到各个桶中：循环原table，注意的是，循环次数为数组table的长度，而不是HashMap的size（键值对的个数）；在循环体中，对原table中任意一个位置，会判断该位置的结点类型：只有一个、链表、红黑树，然后分类进行重哈希。
如果原table的该位置处的桶结构为链表，则该链表会保留元素顺序：它指的是，假如链表中两个结点a和b，有a.next=b或者a.next.next...=b，那么在重哈希以后，如果a和b仍然在同一链表中，依然会保持a.next=b或者a.next.next...=b的关系；当然只是顺序，next的次数与之前是不同的。


```java
/**
 * Initializes or doubles table size.  If null, allocates in
 * accord with initial capacity target held in field threshold.
 * Otherwise, because we are using power-of-two expansion, the
 * elements from each bin must either stay at same index, or move
 * with a power of two offset in the new table.
 *
 * 数组table初始化或者扩容（翻倍）。
 * 如果table为null，则根据初始容量分配。
 * 如果table不为null，由于使用了翻倍的扩展方式，桶里的键值对要么依然在原位置，
 * 要么在原位置再移动2次幂的位置。
 *
 * @return the table
 */
final Node<K,V>[] resize() {
    //oldTab指向原来的table（未调整大小的）
    Node<K,V>[] oldTab = table;
    //原来table的长度
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    //原来table的阈值
    int oldThr = threshold;
    //定义新容量（数组的长度）、新阈值
    int newCap, newThr = 0;
    //如果原table的容量大于0
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {//如果原table的容量已经达到了最大容量值，则不能再扩容了，直接返回
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //进行翻倍（同时保证翻倍后容量值小于最大容量值），并且将阈值也翻倍
        //阈值翻倍？因为阈值的定义为：容量*装载因子；装载因子不变，容量翻倍后，阈值也要翻倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    //如果原table的容量小于等于0，并且原阈值>0
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //此时，已经计算好了新的阈值、新的容量
    //将计算好的新的阈值赋给成员变量threshold
    threshold = newThr;

    //创建一个新的数组，容量为之前计算好的新的容量，并将成员变量table指向这个数组
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;

    //如果原table不为空
    if (oldTab != null) {
        //循环原table，注意的是，循环次数为数组table的长度，而不是HashMap的size（键值对的个数）
        //在循环体中，对原table中任意一个位置，会判断该位置的结点类型：只有一个、链表、红黑树
        //然后分类进行重哈希
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;//该变量保存的是每次循环时的那个结点
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)//也就是原table的该位置处的桶只有一个结点
                    //将结点存储在位置e.hash & (newCap - 1)处（为什么要这么计算？有待研究。。。）
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)//如果原table的该位置处的桶结构为红黑树
                    //重新分布该桶里的结点
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                //如果原table的该位置处的桶结构为链表
                else { // preserve order
                    //上面的preserve order是自带的注释：保留顺序，
                    //它指的是，假如链表中两个结点a和b，有a.next=b或者a.next.next...=b,
                    //那么在重哈希以后，如果a和b仍然在同一链表中，依然会保持a.next=b或者a.next.next...=b的关系
                    //当然只是顺序，next的次数与之前是不同的。


                    //这里的四个变量loHead、loTail、hiHead、hiTail是代表什么呢？
                    //tail和head很容易想到是链表的尾结点和头结点的意思。
                    //看字面lo是lower，hi是high。那么这里的lower和high是什么呢？
                    //这里的高低是指该链表所在的table的原位置与table扩容后链表中将要移动位置
                    //的那些结点所在的新table的新位置。

                    //也就是说该链表中的元素基本会有一半的元素继续待在新table的j处组成新链表m，
                    //这个新链表m的头结点为loHead，尾结点为loTail
                    //另一半元素会移动到新table的j + oldCap位置处组成另一个新的链表n，
                    //这个新链表n的头结点为hiHead，尾结点为hiTail
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;//
                    //对该链表进行循环，从头到尾的顺序
                    do {
                        next = e.next;
                        //链表中的结点进行e.hash & oldCap运算，对于该链表的所有结点而言，
                        //这个结果会大约一半等于0，一半等于1，也就是else的情况
                        if ((e.hash & oldCap) == 0) {//继续留在j处的结点，这些元素在table的该处形成新的链表
                            //这里的逻辑相当于往单链表添加元素（在尾部添加），下面else的逻辑是一样的，只是元素不一样而已。
                            //添加第一个元素a时，此时尾结点为空：则将头、尾结点都指向a
                            //之后再添加元素x时，由于尾结点不再为空，执行loTail.next = x，
                            //即将此时尾结点的属性"下一个结点"设置为x，
                            //再将尾结点指向x。
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {//将会移动到j + oldCap的结点
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    //到这里，两个链表已经形成了，只是还没有指定到它们该在的位置
                    if (loTail != null) {//将其中一个链表的头结点放到新table的j处（当然也是原table的j处）
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {//将另外一个链表的头结点放到新table的j + oldCap处
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

### get

```java
//根据key获取value。
//HashMap中不会包含重复key。也就是对于同一key，只可能存在一个Node。
//如果返回null，并不一定说明不存在包含这个key的键值对，有可能该键值对就是<key，null>，也就是e.value等于null
//可以使用containsKey方法来判断HashMap是否含有某特定key。
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

看方法getNode：

```java
/**
 * Implements Map.get and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @return the node, or null if none
 *
 */
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab;
    Node<K,V> first, e;
    int n;
    K k;//寻找Node过程中的key

    //给定一个键值对，根据这个键值对的key，生成一个哈希值hash，
    //那么这个键值对应该放在table数组的(n - 1) & hash位置处。
    //该位置处可能为空（情况a），
    //可能是只有一个键值对的桶（情况b），
    //可能是链表结构的桶(情况c)，
    //可能是红黑树结构的桶(情况d)

    //数组table不为空 && 数组table的长度大于0 && 根据hash值定位的数组位置处的结点不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //情况b：直接返回该结点
        if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            //情况d：调用红黑树的getTreeNode方法
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //情况c:对单链表进行循环，
            //单链表的查找的平均时间复杂度为为O(n)，n为链表结点个数
            //由于HashMap中链表结构最多有8个结点（更多的话就会这些结点就会转换为红黑树结构），
            //所以很快就可以找到
            do {
                if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    //情况a，返回null
    return null;
}
```

以上是对HashMap中的几个感觉比较重要的方法的分析，如有不对的地方欢迎指正。

