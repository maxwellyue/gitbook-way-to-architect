**1、finally块里的代码是在return之前执行的。**

```
public class Test {
    public static int testFinally() {
        try {
            return 1;
        } catch (Exception ex) {
            return 2;
        } finally {
            System.out.println("execute finally");
        }
    }
    public static void main(String[] args) {
        int result = testFinally();
        System.out.println(result);
    }
}
```

输出结果为：

```
execute finally
1
```
注意：虽然catch块中也有return语句，但是上面的例子中，try中并没有抛出异常，所以不会执行catch块中的内容。

**2、如果try-catch-finally中都有return，那么finally块中的return将会覆盖别处的return语句，最终返回到调用者那里的是finally中return的值。**

```
public class Test {
    public static int testFinally() {
        try {
            return 1;
        } catch (Exception ex) {
            return 2;
        } finally {
            System.out.println("execute finally");
            return 3;
        }
    }
    public static void main(String[] args) {
        int result = testFinally();
        System.out.println(result);
    }
}
```

输出结果为：

```
execute finally
3
```

**3、在try/catch中有return时，在finally块中改变基本类型的数据对返回值没有任何影响；而在finally中改变引用类型的数据会对返回结果有影响。**

```
public class Test {
    public static int testFinally1() {
        int result1 = 1;
        try {
            return result1;
        } catch (Exception ex) {
            result1 = 2;
            return result1;
        } finally {
            result1 = 3;
            System.out.println("execute testFinally1");
        }
    }
    public static StringBuffer testFinally2() {
        StringBuffer result2 = new StringBuffer("hello");
        try {
            return result2;
        } catch (Exception ex) {
            return null;
        } finally {
            result2.append("world");
            System.out.println("execute testFinally2");
        }
    }
    public static void main(String[] args) {
        int test1 = testFinally1();
        System.out.println(test1);
        StringBuffer test2 = testFinally2();
        System.out.println(test2);
    }
}
```

输出结果为：

```
execute testFinally1
1
execute testFinally2
helloworld
```

---
**4、finally块一定会被执行吗？**

不一定，以下两种情况下，finally不会执行：

* try语句没有被执行到。<br>没有进入try代码块，则对应的finally就不会执行。比如，在try语句之前return就返回了，这样finally不会执行；或者在程序进入java之前就出现异常，会直接结束，也不会执行finally块。
* 在try/catch块中有System.exit\(0\)来退出JVM。<br>System.exit\(0\)是终止JVM的，会强制退出程序，finally{}中的代码就不会被执行。


---
本小节内容的组织方式及内容来自：[finally代码块的执行情况](https://www.jianshu.com/p/06755f52ba90)

