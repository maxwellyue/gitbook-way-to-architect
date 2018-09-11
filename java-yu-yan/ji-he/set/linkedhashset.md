# LinkedHashSet

**特性**

* 以插入顺序保存元素，当遍历该集合时候，将会以元素的添加顺序访问集合的元素。
* 迭代性能比HashSet好，但是插入时性能稍微逊色于HashSet（其实是HashMap与LinkedHashMap的特点）。

**实现**

LinkedHashSet的底层数据结构为LinkedHashMap，所有操作均是通过调用LinkedHashMap（继承自HashMap）的方法来实现的。

因为继承自HashSet，而HashSet内部的数据结构为HashMap，所以，LinkedHashSet中的代码只有几个构造器。

```java
public class LinkedHashSet<E> extends HashSet<E> implements Set<E>, Cloneable, java.io.Serializable {

    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }

    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }

    public LinkedHashSet() {
        super(16, .75f, true);
    }

    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }
}
```

父类HashSet中对应的构造器如下

```java
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```



