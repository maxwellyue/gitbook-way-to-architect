#### 不同创建方式的区别

* **使用new关键字来创建。**使用这种方式时，JVM创建字符串对象但不存储于字符串池。我们可以调用intern\(\)方法将该字符串对象存储在字符串池，如果字符串池已经有了同样值的字符串，则返回引用。

* **使用双引号直接创建。**使用这种方式时，JVM去字符串池找有没有值相等字符串，如果有，则返回找到的字符串引用。否则创建一个新的字符串对象并存储在字符串池。

理解下面的题目：

```
1.判断定义为String类型的s1和s2是否相等
    String s1 = "abc";
    String s2 = "abc";
    System.out.println(s1 == s2);                     
    System.out.println(s1.equals(s2));         
2.下面这句话在内存中创建了几个对象?
    String s1 = new String("abc");            
3.判断定义为String类型的s1和s2是否相等
    String s1 = new String("abc");            
    String s2 = "abc";
    System.out.println(s1 == s2);        
    System.out.println(s1.equals(s2));
4.判断定义为String类型的s1和s2是否相等
    String s1 = "a" + "b" + "c";
    String s2 = "abc";
    System.out.println(s1 == s2);        
    System.out.println(s1.equals(s2));
5.判断定义为String类型的s1和s2是否相等
    String s1 = "ab";
    String s2 = "abc";
    String s3 = s1 + "c";
    System.out.println(s3 == s2);
    System.out.println(s3.equals(s2));

解答:
    上面所有equals()方法的结果都是true,因为equals()方法在String类中,我们来看下API中的解释
equals:
    将此字符串与指定的对象比较。当且仅当该参数不为null，并且是与此对象表示相同字符序列的 String 对象时，结果才为true。
    因为String类中字符串是常量；它们的值在创建之后不能更改

第一题中:
    //常量池中没有这个字符串对象,就创建一个,如果有直接用即可
    String s1 = "abc";
    String s2 = "abc";
    System.out.println(s1 == s2);             //==号比较的是地址值,true    
    System.out.println(s1.equals(s2));         //比较的是字符串的内容:true
第二题:
    //创建几个对象
    //创建两个对象,一个在常量池中,一个在堆内存中
    String s1 = new String("abc");        
    System.out.println(s1);
第三题:
    String s1 = new String("abc");            //记录的是堆内存对象的地址值        
    String s2 = "abc";                        //记录的是常量池中的地址值
    System.out.println(s1 == s2);             //false
第四题:
    //byte b = 3 + 4;                        //在编译时就变成7,把7赋值给b,常量优化机制
    String s1 = "a" + "b" + "c";            //java中有常量优化机制,在编译时期就能确定s2的值为"abc",所以编译时期,在常量池中创建"abc"
    String s2 = "abc";                        //执行到这里时常量池中已经有了"abc",所以就不再创建,所以s1和s2指向的是常量池中同一个字符串常量"abc"
    System.out.println(s1 == s2);             //true,java中有常量优化机制    
第五题:
    String s1 = "ab";
    String s2 = "abc";
    String s3 = s1 + "c";                    
    System.out.println(s3 == s2);    
    
    String相加的时候，底层是通过StringBuiler的append和toString来完成的，而toString是通过构造函数来实现的 
    也就是相当于是一个new的操作，所以为false。但是append操作的时候是在同一个对象上面操作的，因此引用所指向的地址是会发生改变的         
```

---

内容来源：

[Java中String类的常见面试题](https://www.jianshu.com/p/44224e650520)

[【java】String类常见面试题](http://blog.csdn.net/lzm18064126848/article/details/53839535)