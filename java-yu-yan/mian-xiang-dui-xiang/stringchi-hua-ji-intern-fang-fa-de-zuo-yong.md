> 声明：以下内容可能有误

# 常量池

要理解String的池化，必须首先知道**常量池**这个概念。

* 方法区：存储Class文件编译后的类信息、常量、静态变量、编译后的代码
* 常量池：存储Class文件编译后的字面量和符号引用；但这个常量池又叫运行时常量池，则说明可以在程序运行期间将新的常量放入池中，如使用String类的intern\(\)。那么为什么要使用常量池：通过**对象共享**的方式避免频繁的创建和销毁对象，从而节省空间，且节省运行时间（比如比较字符串时，==比equals\(\)快。对于两个引用变量，只用==判断引用是否相等，也就可以判断实际值是否相等）。

> 通俗的来说，Class文件中除常量表（constant pool table）之外的信息都会加载到方法区，而常量表中的信息都会加载到常量池中。

在JDK1.6及之前，常量池位于方法区中，而在JDK1.7及之后，常量池位于堆中。

# String

String是不可变类，即一旦创建，就不能再次修改这个对象。而在编程中，String类型是最长用到的数据类型，对字符常量池化后，可以大大减少字符的数量，从而节省空间。

比如，通过如下方式声明两个String类型的变量，只会在内存中（常量池中）创建一个“abc”这个字符串，而s1和s2均会指向这个地址，所以，无论使用s1==s2还是s1.equals\(s2\)，都会得到true的结果。

```java
String s1 = "abc";
String s2 = "abc";
System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
//输出结果
true
true
```

但是，如果使用new关键字来创建String对象，情况则会不同：

```java
String s1 = new String("abc");    ------①
String s2 = new String("abc");    ------②
System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
//输出结果
fasle
true
```

对于`new String("abc")`这句代码需要这么来理解：在类加载的时候，就会将“abc”放在常量池中；在代码真正执行的时候，会从常量池中复制“abc”到堆中或者说在堆中创建“abc”这个字面量（如下面的字节码所示）。通过new这种方式创建String对象，与使用new创建其他普通对象是没有区别的，上述代码中的①和②均会在堆中创建对象，只不过这两个对象的内容都是“abc”，但内存地址不一样，所以，使用==来比较，结果为false。

```java
0: new  #2; //class java/lang/String  
3: dup  
4: ldc  #3; //String abc
6: invokespecial    #4; //Method java/lang/String."<init>":(Ljava/lang/String;)V  
9: astore_1
```

前面说到，可以在运行期间动态地向常量池中添加常量，如果想要在运行期间向运行池中添加一个字符串，则可以使用String类中提供的一个方法：`intern()`，`intern`这个单词是“驻留”的意思，也就是将某个字符串放置到常量池中。但是，如何放，不同JDK版本是不一样的（因为常量池的位置发生了变化）。

* JDK1.6及之前：如果常量池中已经存在想要放入的字符串（`equals`），则直接返回已经存在的那个字符串的地址，如果不存在，则将其放入，并返回引用地址。

* JDK1.7及之后：如果常量池中不存在，但堆中存在，则直接在常量池中存储这个字符串在堆中的地址，而不是像JDK1.6那样存储这个字符串本身。

即通过intern\(\)方法，可以确保从常量池中得到与想要放入的字符串值（假设为s）equals的一个字符串的地址（假设为s\_addr），并且只要s是equals的，那么得到的永远是这个s\_addr。

使用intern的场景：需要使用new String\(String s\) 大量创建String的对象的时候，使用new String\(s.intern\(\)\) 来替代，可以节省内存空间。

TODO

关于intern\(\)还有没能理解的地方，比如在JDK1.8中：执行以下代码，输出结果为fasle和true。

```java
String s = new StringBuilder("ja").append("va").toString();
System.out.println(s.intern() == s);

String s1 = new StringBuilder("abc").append("abc").toString();
System.out.println(s1.intern() == s1);
```

# 参考

[请别再拿“String s = new String\("xyz"\);创建了多少个String实例”来面试了吧](http://rednaxelafx.iteye.com/blog/774673)

[Java常量池理解与总结](https://www.jianshu.com/p/c7f47de2ee80)

[String池化及intern方法的作用](https://darrenyjy.github.io/2016/05/28/String%E6%B1%A0%E5%8C%96%E5%8F%8Aintern%E6%96%B9%E6%B3%95%E7%9A%84%E4%BD%9C%E7%94%A8/)

[深入解析String\#intern](https://tech.meituan.com/in_depth_understanding_string_intern.html)

[Java技术——你真的了解String类的intern\(\)方法吗](https://blog.csdn.net/seu_calvin/article/details/52291082)

  




