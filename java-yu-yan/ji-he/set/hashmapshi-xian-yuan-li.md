# HashSet

**概述**

HashSet实现Set接口，由哈希表（实际上是一个HashMap实例）支持。

* 不保证迭代顺序，特别是它不保证该顺序恒久不变
* 允许使用null元素

HashSet基于HashMap实现的，HashSet底层使用HashMap来保存所有元素，因此HashSet的实现比较简单，相关HashSet的操作，基本上都是直接调用底层HashMap的相关方法来完成。

HashSet的源码如下（去除无关代码）：

```java
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable {
    
    //底层数据结构
    private transient HashMap<E,Object> map;

    //定义一个虚拟的Object对象作为HashMap的value，ji
    private static final Object PRESENT = new Object();

    //默认的HashMap的容量为16，装载因子为0.75
    public HashSet() {
        map = new HashMap<>();
    }

    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }

   
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    //仅用于LinkedHashMap，包私有的，用户无法调用
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }

    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }

    public int size() {
        return map.size();
    }

    public boolean isEmpty() {
        return map.isEmpty();
    }

    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }

    //清空Set
    public void clear() {
        map.clear();
    }

    //浅拷贝，Set中的元素并未被拷贝
    @SuppressWarnings("unchecked")
    public Object clone() {
        try {
            HashSet<E> newSet = (HashSet<E>) super.clone();
            newSet.map = (HashMap<E, Object>) map.clone();
            return newSet;
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }
}
```

可以看出，HashSet的源码非常简单，因为复杂性都交给了HashMap：将所有元素作为内部HashMap的key插入，而内部HashMap的所有key的value均为一个static final的对象。



