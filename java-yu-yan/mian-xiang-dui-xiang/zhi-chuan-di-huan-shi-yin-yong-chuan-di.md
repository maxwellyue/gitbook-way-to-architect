# 值传递还是引用传递？

值传递/引用传递，描述的是将参数传给方法时，是如何对这个参数进行求值的，而不是值参数的内容的数据类型。

**值传递（pass by value）**

复制一份传递进来的值，无论传递进来的值是基本类型还是引用类型，都会进行复制，即**方法得到的是所有参数值的一个拷贝，**不能修改传递给它的任何参数的内容**。**

**引用传递\(pass by reference\)**

使用传递进来的参数的地址，Java中不存在引用类型。

可以这么来理解这两者的区别：对于一个对象来讲，我们把对象本身称为“原件”，对象的副本是“复印件”，值传递是指参数传递的时候其实只是获取到了被传者的一个副本即“复印件”，而引用传递获取到的是被传者本身也即“原件”。

对Java来言，方法参数传递只有一种，就是 “**pass-by-value**”，也就是**值传递**。

* 如果是基本类型，则将原有的数据拷贝一份，方法内的操作对原有的数据不会有影响。
* 如果是引用类型，则将这个引用类型的变量拷贝一份，“复印件”也是引用类型，且与“原件”指向同一对象。

即在Java中**不管参数的类型是什么，一律传递参数的副本。**

请使用下述示例检验是否真正地明白Java是值传递。

```java
//示例1
public static void main(String[] args) {
    List<Integer> list = new ArrayList<Integer>();
    list.add(1);
    list.add(2);
    add(list);
    list.forEach(item ->{
        System.out.println(item);
    });
}

static void add(List<Integer> list) {
    list.add(3);
}
//输出如下
1
2
3


//示例2
public static void main(String[] args) {
    String a = "A";
    append(a);
    System.err.println(a);
}

static void append(String str) {
    str += "is a";
}
//输出如下
A


//示例3
public static void main(String[] args) {
    int num = 5;
    addNum(num);
    System.err.println(num);
}

static void addNum(int a) {
    a = a + 10;
}
//输出如下
5
```

## 参考

[Java 到底是值传递还是引用传递？](https://www.zhihu.com/question/31203609)

[What's the difference between passing by reference vs. passing by value?](https://stackoverflow.com/questions/373419/whats-the-difference-between-passing-by-reference-vs-passing-by-value)

