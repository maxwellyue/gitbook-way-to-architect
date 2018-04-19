# Integer.parseInt\(\)与Interger.valueof\(\)

下面的代码有什么区别：

```text
Integer.parseInt("12");
Integer.valueOf("12");
```

看源码：

```text
public static int parseInt(String s) throws NumberFormatException {
        return parseInt(s,10);
}
public static Integer valueOf(String s) throws NumberFormatException {
        return Integer.valueOf(parseInt(s, 10));
}
```

可以看出两者的区别：

返回值不同，parseInt\(String s\)返回的是int，而valueOf\(String s\)返回的是Integer

所以，一般用`Integer.parseInt(str)`，除非你要返回`Integer`类型，不然还有封装拆箱，性能多少会耗费些。

