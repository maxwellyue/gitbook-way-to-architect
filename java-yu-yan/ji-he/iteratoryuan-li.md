

## Iteratable

在Java中，容器可进行迭代操作，使用Iteratable接口来标识：

Iteratable中的接口如下：

```java
public interface Iterable<T> {
    
    //返回迭代器
    Iterator<T> iterator();

    //对每个元素执行action操作，提供了默认实现
    //@since 1.8
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    //返回Spliterator
    //@since 1.8
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

Collection接口继承了Iterable接口，这表明所有的Colection均支持迭代操作：

```java
public interface Collection<E> extends Iterable<E> {... ...}
```

foreach示例代码

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1,3,5,7));
list.forEach(element ->{
    System.out.println(element);
});

//输出如下
1
3
5
7
```

# Iterator

Iterator，迭代器，用来遍历Collection中的元素。对于Collection的所有实现类而言，必须实现以下接口，即必须返回一个迭代器：

```java
Iterator<E> iterator();
```

Iterator中的接口如下：

```java
public interface Iterator<E> {

    //判断是否下一个元素
    boolean hasNext();

    //返回下一个元素
    E next();

    //移除元素
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    //对剩余元素按顺序执行某个操作
    @since 1.8
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

主要原理

* 创建指向某个`Collection`的`Iterator`对象时，`Iterator`内部的游标指向的第一个元素。
* 调用`hasNext()`的时候，只是判断下一个元素的有无，并不移动游标。
* 调用`next()`的时候，返回游标指向的元素，之后向下移动游标，使其指向Collection中的下一个元素。
* 调用`remove()`，删除当前游标上次指向的元素

```java
List<Integer> list = new ArrayList<>(Arrays.asList(6, 2, 2, 6, 8));
Iterator<Integer> iterator = list.iterator();
while (iterator.hasNext()){
    Integer next = iterator.next();
    System.out.println(next);
    if(next == 2){
        iterator.remove();
    }
}
System.out.println(">>>>>>>>>after");
for (Integer integer : list){
    System.out.println(integer);
}

//输出如下
6
2
2
6
8
>>>>>>>>>after
6
6
8
```

### **ArrayList中的实现**

ArrayList中有两个Iterator的实现，一个Itr，另一个是增强的ListItr（提供了向前移动/遍历的功能，是ListIterator的实现），这里以Itr为例。

```java
public Iterator<E> iterator() {
    return new Itr();
}
```

```java
private class Itr implements Iterator<E> {
        int cursor;       // 返回的下一个元素的下标，这个下标就是ArrayList中的下标
        int lastRet = -1; // 上次返回的元素的的下标
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;//将游标指向下一个元素
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();
            try {
                ArrayList.this.remove(lastRet);//这里会实际删除ArrayList中的元素，会改变ArrayList的实际长度
                cursor = lastRet;//所以，让游标前移一位，相当于cursor = cursor - 1；
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        //暂时忽略该方法
        public void forEachRemaining(Consumer<? super E> consumer) {
            ... ...        
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

可知，在ArrayList内部的Iterator实现中，内部维护了一个游标cursor，该cursor初始化为0，每调用一次next\(\)方法时，在返回cursor指向的元素之后，将curosr值加1；同时，使用变量lastRet来记录上次next\(\)方法返回的元素的下标。在调用remove\(\)方法后，由于实际数据量少了1个，所以将cursor指向lastRet，以便以便下次next\(\)调用时可以返回正确的结果。

同时，也可以发现：在实现`hasNext()/next()/remove()`等方法时，均是直接操作或依赖ArrayList底层的数据结构。

在Collection其他子类中的实现中，也是如此。由于Collection中各实现类底层结构不尽相同，遍历时所采用的方式也就不一样，所以每个实现类一般都会各个维护自身的Iterator实现。

# 





## 参考

[迭代器iterator原理和设计模式](https://blog.csdn.net/hebixi/article/details/52075684)

  








