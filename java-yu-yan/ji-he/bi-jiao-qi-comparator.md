# 比较器

在Java中，如果想要对对象进行排序，一般有两种方式：

* 待比较对象的类实现Comparable接口，重写compareTo\(T o\)方法

* 自定义Comparator，重写Compare\(T o1, T o2\)方法

## **Comparable**

若一个类实现了Comparable接口，就意味着“**该类支持排序**”。

* 对实现Comparable接口的类的对象的List/数组，则该List/数组可以通过Collections.sort/ Arrays.sort进行排序。
* 对实现Comparable接口的类的对象，可以用作有序键值对容器\(如TreeMap\)中的键或有序集合\(如TreeSet\)中的元素，而不需要指定比较器。

它的定义如下：

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

compareTo\(T o\)的返回结果的说明：假设我们通过 x.compareTo\(y\) 来“比较x和y的大小”。

* 若返回“负数”，意味着“x比y小”
* 返回“零”，意味着“x等于y”
* 返回“正数”，意味着“x大于y”

## **Comparator**

Comparator的定义如下：

```java
public interface Comparator<T> {

    int compare(T o1, T o2);

    boolean equals(Object obj);
}
```

说明：

* 若一个类实现Comparator接口，则必须要实现compareTo\(T o1, T o2\) 函数，但可以不实现 equals\(Object obj\) 函数。

> 为什么可以不实现 equals\(Object obj\) ：  Java中的一切类都是继承于java.lang.Object，在Object.java中实现了equals\(Object obj\)函数，所以，其它所有的类也相当于都实现了该函数。即任何类默认都是已经实现了equals\(Object obj\)的。

* compare\(T o1, T o2\) 的返回结果的说明：

  * 返回“负数”，意味着“o1比o2小”；
  * 返回“零”，意味着“o1等于o2”；
  * 返回“正数”，意味着“o1大于o2”。

  > 注意：**JDK1.7中，必须返回一对相反数，如1和-1，不能是1和0。因为1.7的排序算法采用**[**TimSort**](https://baike.baidu.com/item/TimSort)**，对返回值有严格要求**

无论对于Comparable还是Comparator，都有这样一个推荐做法：

```java
(compare(x, y)==0) ==  x.equals(y)
x.compare(y) == x.equals(y)
```

即两个对象通过compare比较等于0， 则最好这两个对象是equals的。反之，如果两个对象通过compare比较不相等，那么这两个对象通过equals比较时，返回fasle。

## 使用

待排序的对象的类

```java
public class Person implements Comparable<Person>{

    private String name;

    private int age;

    public Person(String name, int age){
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Person o) {
        int res = 0;
        if(age > o.age){
            res = 1;
        }else if(age < o.age){
            res = -1;
        }
        return res;
    }

    @Override
    public String toString() {
        return String.format("%s: %d", this.name, this.age);
    }

    //省略getter和setter方法
}
```

排序

```java
public class Example {

    public static void main(String[] args) {

        //初始化对象集合
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("a", 18));
        personList.add(new Person("aa", 6));
        personList.add(new Person("aaa", 32));
        personList.add(new Person("aaaa", 21));

        System.out.println("默认按年龄排序");
        Collections.sort(personList);
        personList.forEach(person -> {
            System.out.println(person);
        });


        System.out.println("自定义按名字的长度排序");
        Collections.sort(personList, new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                int length1 = o1.getName().length();
                int length2 = o2.getName().length();
                int res = 0;
                if (length1 > length2) {
                    res = 1;
                } else if (length1 < length2) {
                    res = -1;
                }
                return res;
            }
        });
        personList.forEach(person -> {
            System.out.println(person);
        });
    }
}
//输出如下
默认按年龄排序
aa: 6
a: 18
aaaa: 21
aaa: 32
自定义按名字的长度排序
a: 18
aa: 6
aaa: 32
aaaa: 21
```



