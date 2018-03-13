## static

---

#### static修饰变量

* 声明为static的变量实质上就是全局变量。

* 所有此类实例共享此静态变量，也就是说在类装载时，只分配一块存储空间，所有此类的对象都可以操控此块存储空间。

* 一般来说，使用`类名.static变量`来调用或修改该变量。

* 一般是public static final 来定义静态常量。

#### static修饰方法

* 好处：无需本类的对象即可调用此方法，直接用`类名.static方法`的方式使用。（当然也可以用本类的对象调用，但是不推荐这么做）

* 限制：static方法内部不能调用非static方法，也不能使用非static数据（这里指的是方法内部，形参除外），不能以任何方式引用this或super。

* 有两种情况需要使用静态方法：

  一个方法不需要访问对象状态，其所需参数都是通过显式参数提供；

  一个方法只需要访问类的静态域；

#### static修饰代码块

* 用static修饰的代码块只会在类进行初始化的时候调用一次，而非static修饰的代码块则是在每次生成对象的时候都进行一次调用。

#### static修饰内部类

* 普通类不可以用static修饰，只有内部类可以。
* 普通的内部类对象隐含地保存了一个引用，指向创建它的外围类对象。
* 声明为static的内部类，不依赖于外部类实例而被实例化，不持有外部类的引用。
* 声明为static的内部类，不能访问外部类的非静态成员\(包括非静态变量和非静态方法\)，只能访问外部类中的static成员。



## final

---
final 关键字有三个东西可以修饰的。修饰类，方法，变量。  详细解释一下：

#### final修饰类

使用了 final 的类不能再派生子类，就是说不可以被继承了。有些 java 的面试题里面，问 String 可不可以被继承。答案是不可以，因为 java.lang.String是一个 final 类。这可以保证 String 对象方法的调用确实运行的是 String 类的方法，而不是经其子类重写后的 方法。

#### final修饰方法

被定义为 final 的方法不能被重写了，如果定义类为 final 的话，是所有的方法都不能重写。而我们只需要类中的某几个方法，不可以被重写，就在方法前加 final 了。而且定义为 final 的方法执行效率要高的啊。

#### final修饰变量

这样的变量就是常量了，在程序中这样的变量不可以被修改的。修改的话编译器会抱错的。而且执行效率也是比普通的变量要高。final 的变量如果没有赋予初值的话，其他方法就必需给他赋值，但只能赋值一次。


## 为啥static和final老是在一起？
---
final是限制不可变，而static是让变量在多个对象实例中一致。并不是一定要同时使用static和final，但往往的情况是，你需要一个在多个对象实例中一致的变量，而且还不想让其它的类去修改这个变量，那么此时就需要static和final同时出现。

final与static final的区别是：final在一个对象实例中唯一，static final在多个对象中都唯一；

更多是出于设计设计考虑。别人一看这个关键字，就知道这个成员大概能起到什么作用，更快的明白你程序的架构，这就是“语义”。


---
内容来源：

[Java中static关键字的用法](https://www.jianshu.com/p/b6d2d81a5991)

[static和final的区别](https://www.jianshu.com/p/9d4a41df164f)

[Java里为什么要用final关键字？static为什么要和final一起用？](https://segmentfault.com/q/1010000002684166)
