# ArrayList

ArrayList是最常用的集合之一，其特点为：

* 数据结构为数组，随机访问快，插入或删除慢
* 线程不安全
* 元素可为空，可重复，有序（是指插入的顺序）

下面，就从以上特点入手，简单分析一下ArrayList的代码。

## 数据结构为数组

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
    
    //默认容量大小：10，即如果在创建ArrayList的时候如果不指定初始大小，则默认为10
    private static final int DEFAULT_CAPACITY = 10;
    
    //底层存放元素的数据结构，就是一个数组
    transient Object[] elementData; 
    
    //ArrayList中当前的元素个数
    private int size;

    /**
     *
     * 构造方法，指定初始大小。
     *
     * 如果initialCapacity大于0，就new一个数组，这个数组的容量为initialCapacity
     * 如果initialCapacity等于0，就让底层数组为{}。
     * 如果initialCapacity小于0，就抛出异常。
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
        }
    }

    //构造方法，初始大小为10
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    //构造方法，传入一个集合
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
}
```

## 随机访问快

```java
//根据位置获取元素：就是根据数组下标获取的
public E get(int index) {
    rangeCheck(index);
    return elementData(index);
 }
```

在数组中的元素，可以通过下标直接访问，因此说随机访问速度快。

此外，ArrayList还实现了**RandomAccess接口**：该接口仅仅是一个标识，其中并没有具体的方法要重写。实现该空接口就意味着支持随机访问，并且在遍历的时候：

```java
for (int i=0, n=list.size(); i < n; i++)
         list.get(i);

runs faster than this loop:

for (Iterator i=list.iterator(); i.hasNext(); )
         i.next();

```

也就是说，在遍历集合的时候，可以通过这种方法来优化：

```java
if (list instanceof RandomAccess) { 
        for (int i=0; i<size; i++)
                //....
} else {
        for (Iterator i=list.iterator(); i.hasNext(); )
               //...
}
```

## 插入或删除速度慢

**首先看add方法：**

```java
//添加元素：放在最后（就是数组下标从0到length依次填充）
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}


//在某个位置添加元素，如果该位置以及该位置的右边含有元素，原有元素就会右移。
//要注意与set(int index, E element)方法的区别
public void add(int index, E element) {
    rangeCheckForAdd(index);
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

对于`add(E e)`方法，会将元素添加到数组的最后，此时速度并不慢。但是在调用`add(int index, E element)`方法的时候，会将index位置以及右侧所有元素向右移动一位，因此说插入速度慢。  
此外，如果元素为空，也会成功添加到集合中，也就是说ArrayList允许元素为空，也可以重复。

  
注意`add(int index, E element)`与`set(int index, E element)`方法的区别：

* add并不会改变原index位置处的元素，而是index位置及该位置右边的元素右移一位，将新元素插在index位置处。
* set则是用新元素直接替换掉index位置的旧元素，但不会影响index位置之后的元素。

**再看移除操作：**

```java
//移除某位置的元素，该元素右边的元素都会左移
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

同样是将元素左移，需要操作大量除index位置处的元素。

在上面的代码中，元素左移和右移是通过方法**`System.arraycopy`**方法实现的。`arraycopy`是`System`类的静态方法，是一个native方法。

```java
 / * @param      src      the source array.
   * @param      srcPos   starting position in the source array.
   * @param      dest     the destination array.
   * @param      destPos  starting position in the destination data.
   * @param      length   the number of array elements to be copied.
 */
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

意思是：将数组src从位置srcPos开始的length个元素，放到数组dest中（从位置destPos开始放）。

## 线程不安全

可以先看[ArrayList线程不安全分析](https://link.jianshu.com/?t=https://my.oschina.net/u/656588/blog/665259)中的测试小例子，理解几种线程不安全的情况。我的理解是，多个线程对同一ArraryList实例进行修改时（或者一些线程在修改，另一些线程在遍历），由于未进行同步，导致成员变量size值或者内部数组elementData元素不一样，导致出错：或者越界或者遍历时fail-fast等。限于水平，对Java内存模型的认识不足，不再叙述。

## 扩容方式

什么时候会扩容？肯定是在添加元素的时候，如果内部数组不足以存放要添加的元素，才会去扩大数组的容量。看下面的代码（去掉了无关的代码）

```java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  
        elementData[size++] = e;
        return true;
}

//这里minCapacity就是size + 1，也就是内部数组在调整后至少要这么大。
private void ensureCapacityInternal(int minCapacity) {
        //如果此时内部数组为空数组，就把minCapacity设置为默认容量和minCapacity之间的大者
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }

public void ensureCapacity(int minCapacity) {
        //minExpand字面意思就是要扩展的最小值，
        //如果此时内部数组为空数组，minExpand为默认值10，否则为0
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;
        //除非是空数组，否则都会执行下面的逻辑
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
}
        
private void ensureExplicitCapacity(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
}

//扩容操作，使得ArrayList至少可以放minCapacity个元素
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //真正的扩容语句：位操作，右移相当于除以2
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
}
```

扩容的大小为

```java
newCapacity = oldCapacity + (oldCapacity >> 1) = 1.5 * oldCapacity
```

如果一开始就能估计出ArrayList的大小，构造ArrayList时指定初始容量是很好的习惯，可以避免多次扩容带来的额外开销。  
  








