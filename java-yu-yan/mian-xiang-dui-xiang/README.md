# 面向对象

Java中是采用值传递：

* 如果参数是基本类型，传递的是基本类型的字面量值的拷贝。
* 如果参数是引用类型，传递的是该参量所引用的对象在堆中地址值的拷贝。

检验一下是不是真的理解了。

下面程序的输出结果是：

```text
public static void main(String[] args) {
    String x = new String("ab");
    change(x);
    System.out.println(x);
}

public static void change(String x) {
     x = "cd";
}
```

输出为：ab

纠结概念有时候让人很烦恼，重要的是明白传的是什么，不要纠结于“引用的拷贝”是该叫引用还是叫值这种概念上。

更多解释：

[Java 到底是值传递还是引用传递？](https://www.zhihu.com/question/31203609)

