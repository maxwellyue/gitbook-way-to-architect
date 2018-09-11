# Copy-On-Write容器

**概念**

Copy-On-Write简称COW，即写时复制，是一种程序设计中的优化策略。其基本思路是：从一开始大家都在共享同一个内容，当某个人想要修改这个内容的时候，才会真正把内容Copy出去形成一个新的内容然后再改，这是一种延时懒惰策略。

当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种**读写分离**的思想，读和写不同的容器，体现的是一种**以空间换时间**的思想。

```java
//add的时候，使用重入锁，保证只有一个线程在修改
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            //获取当前数组
            Object[] elements = getArray();
            int len = elements.length;
            //copy当前数组到newElements
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            //将元素添加到newElements
            newElements[len] = e;
            //将集合中的底层数组修改为指向newElements
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
}
//get并未使用任何锁，所以在一个线程add的同时，其他线程均可以进行get操作
public E get(int index) {
        return get(getArray(), index);
}
```

**适用场景**

读多写少

**问题**

* **内存占用大**：在进行写操作的时候，内存里会同时驻扎两个对象的内存。如果这些对象占用的内存比较大，很有可能造成频繁的Yong GC和Full GC。针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制。
* **数据只能保证最终一致性，非实时一致性**：所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。

# Copy-On-Write实现类

**CopyOnWriteArrayList**

* `java.util.ArrayList`的线程安全版本：所有的修改操作都是通过对底层数组的最新copy来实现。

* 允许null值：`All elements are permitted, including null`

**CopyOnWriteArraySet**

* 所有操作在内部使用`CopyOnWriteArrayList`的`java.util.Set`。

* set元素数量少，只读操作远多于修改操作，在遍历的时候需要排除其他线程的干扰。

无论是CopyOnWriteArrayList还是CopyOnWriteArraySet，由于都是采用的写时复制的思想，所以要注意以下几点：

* **只适合读多写少的场景**，因为修改操作（`add`、`set`、`remove`等）的代价非常昂贵：需要对整个底层数组进行copy。

* **关于迭代操作需要注意的地方**

* * 迭代方法使用的是迭代开始时的数组的引用，这个数组在该迭代器迭代的整个过程中都不会改变，所以该接口和迭代器不会抛出`ConcurrentModificationException`。
  * 迭代器不会反应出该迭代器创建之后对应list的所有增加、移除、修改等操作。
  * 迭代器自身的元素改变方法，如`remove`、`set`和`add`都是不支持的。

  
  



