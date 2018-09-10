# TreeMap

TreeMap的使用场景：当想要按照键值对中的key的大小来对map中的键值对进行排序的时候。

TreeMap继承AbstractMap，实现NavigableMap：AbstractMap表明TreeMap是个key-value的容器， NavigableMap继承自SortedMap，表明TreeMap支持排序。

```java
public class TreeMap<K,V> extends AbstractMap<K,V> implements NavigableMap<K,V>

public interface NavigableMap<K,V> extends SortedMap<K,V>
```

## 主要属性

```java
//比较器，TreeMap排序的规则，如果为空，则使用key的自然顺序进行排序
private final Comparator<? super K> comparator;
//节点的数据结构（即红黑树的结点）
private transient Entry<K,V> root = null;
//大小或容量
private transient int size = 0;

//Entry的数据结构
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
}
```

TODO

