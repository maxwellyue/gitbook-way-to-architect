# 集合

会出问题，原因：

Iterator 是工作在一个独立的线程中，并且拥有一个 mutex 锁。 Iterator 被创建之后会建立一个指向原来对象的单链索引表，当原来的对象数量发生变化时，这个索引表的内容不会同步改变，所以当索引指针往后移动的时候就找不到要迭代的对象，所以按照 fail-fast 原则 Iterator 会马上抛出 java.util.ConcurrentModificationException 异常。

所以 Iterator 在工作的时候是不允许被迭代的对象被改变的。但你可以使用 Iterator 本身的方法 remove\(\) 来删除对象， Iterator.remove\(\) 方法会在删除当前迭代对象的同时维护索引的一致性。

错误方式：

```text
public class ListRemoveTest {

    public static void main(String[] args) {

        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 4, 6, 8));
        Iterator<Integer> iterator = list.iterator();
        for (; iterator.hasNext(); ) {
            int element = iterator.next();
            if (element == 2) {
                list.remove(element);//这里不能使用list来删除元素
            }
        }
    }

}
```

正确方式1：

```text
public class ListRemoveTest {

    public static void main(String[] args) {

        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 4, 6, 8));
        Iterator<Integer> iterator = list.iterator();
        for (; iterator.hasNext(); ) {
            int element = iterator.next();
            if (element == 2) {
                iterator.remove();
            }
        }
    }

}
```

正确方式2：

```text
public class ListRemoveTest {

    public static void main(String[] args) {

        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 4, 6, 8));
        for (int i = 0; i < list.size(); i++ ) {
            int element = list.get(i);
            if (element == 2) {
                list.remove(i);
            }
        }

    }

}
```

