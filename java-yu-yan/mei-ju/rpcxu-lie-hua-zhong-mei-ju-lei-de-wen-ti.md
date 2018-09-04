# 枚举类在序列化中的问题

序列化的时候，仅仅是将**枚举对象的name属性**输出到结果中，反序列化的时候则是通过**java.lang.Enum的valueOf方法**来根据名字查找枚举对象。

同时，编译器是不允许任何对这种序列化机制的定制的，因此禁用了writeObject、readObject、readObjectNoData、writeReplace和readResolve等方法。

假定服务端进行序列化，客户端进行反序列化，假定原来使用相同的API版本，但发生了以下情况：

**1、服务端枚举多了一个枚举值**

```java
//服务端
public enum Color {
  Red,
  Blue,
  Green;
}
//客户端
public enum Color {
  Red,
  Blue
}
```

如果服务端返回一个Color.Green给客户端，此时反序列化调用枚举类的valueOf方法来获取反序列化，但是客户端的枚举类中没有Green，那么客户端反序列化会直接抛出异常。

**2、服务端枚举ordinal值以及枚举类成员变量值和客户端不一致**    
假设服务端的枚举类为

```java
//服务端
public enum Color {
    RED("red"),WHITE("white"),BLACK("black");

    private String value;

    Color(String value) {
        this.value = value;
    }
}
//客户端
public enum Color {
    RED("red"),BLACK("xblack");

    private String value;

    Color(String value) {
        this.value = value;
    }
}
```

假如服务端传递给客户端Color.BLACK\(ordinal为2，对应的value为black\)，此时客户端拿到的Color.BLACK对应的ordinal为1，对应的value为xblack。

还有一点要特别注意：**枚举是单例的**！如下代码所示：

```java
//定义枚举类
public static enum Color{
    WHITE("white"),
    BLACK("black");

    public String value;

    Color(String value) {
        this.value = value;
    }

    public void setValue(String value){
        this.value = value;
    }
}
//测试
public class EnumExample {

    @Test
    public void enumTest(){
        Color color1 = Color.WHITE;
        Color color2 = Color.WHITE;

        color1.setValue("val1");
        color2.setValue("val2");

        System.out.println(color1.value);
        System.out.println(color2.value);
    }
}
//输出如下
val2
val2
```

**总结**

* 尽量不要在RPC的接口中使用枚举类了，直接使用public static final的方式，除非这个枚举类以后确实是不会修改的。
* 在枚举类中使用字符串时直接使用name\(\)就行，不要再做过度封装，尽量保持枚举类的简洁
* 枚举类使用在RPC接口上的时候就一定要小心，重构的时候要注意保持ordinal
* 枚举在序列化和反序列化的时候，除了name值，其他啥都不带的

## 参考

[枚举在hessian序列化和反序列化中的问题](http://yangbolin.cn/2016/05/22/enum-probolems-in-hessian/)

[Enum反序列化问题](http://xiaobaoqiu.github.io/blog/2015/04/01/enumfan-xu-lie-hua-wen-ti/)

