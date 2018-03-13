## 值传递还是引用传递？

---

Java中是采用值传递：

* 如果参数是基本类型，传递的是基本类型的字面量值的拷贝。

* 如果参数是引用类型，传递的是该参量所引用的对象在堆中地址值的拷贝。



检验一下是不是真的理解了。

下面程序的输出结果是：

```
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





---

更多解释：  

[Java 到底是值传递还是引用传递？](https://www.zhihu.com/question/31203609)

