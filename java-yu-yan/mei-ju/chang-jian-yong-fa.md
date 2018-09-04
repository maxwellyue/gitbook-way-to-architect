# 常见用法

### 常见用法

**判断两个枚举值相等是用==还是equal\(\)：**直接使用==，因为它的equal\(\)方法已经重写为了==，如下：

```java
public final boolean equals(Object other) {
    return this==other;
}
```

**根据常量获取枚举值**

```java
//定义枚举类
public enum Color{
    RED, WHITE
}
//获取枚举值
Color red = Color.valueOf("RED");
//获取所有枚举值
Color[] values = Color.values();
//获取枚举值的name
String name = Color.RED.name();
System.out.println(name);//输出:RED
//获取枚举值在类中出现的顺序
int ordinal = Color.WHITE.ordinal();
System.out.println(ordinal);//输出:1
```

### 使用场景

1、**定义常量**

在JDK1.5 之前，我们定义常量都是： public static fianl…. 。现在好了，有了枚举，可以把相关的常量分组到一个枚举类型里，而且枚举提供了比常量更多的方法。

```java
public enum Color {  
  RED, GREEN, BLANK, YELLOW  
}
```

**2、switch判断**

使用枚举，能让我们的代码可读性更强。

```java
enum Signal {  
    GREEN, YELLOW, RED  
}  
public class TrafficLight {  
    Signal color = Signal.RED;  
    public void change() {  
        switch (color) {  
        case RED:  
            color = Signal.GREEN;  
            break;  
        case YELLOW:  
            color = Signal.RED;  
            break;  
        case GREEN:  
            color = Signal.YELLOW;  
            break;  
        }  
    }  
}
```

**3、向枚举中添加新方法**

如果打算自定义自己的方法，那么必须在enum实例序列的最后添加一个分号。而且 Java 要求必须先定义 enum 实例。

```java
public enum Color {  

    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    // 普通方法  
    public static String getName(int index) {  
        for (Color c : Color.values()) {  
            if (c.getIndex() == index) {  
                return c.name;  
            }  
        }  
        return null;  
    }  
    // get set 方法  
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public int getIndex() {  
        return index;  
    }  
    public void setIndex(int index) {  
        this.index = index;  
    }  
}
```

**4、覆盖枚举的方法**

下面给出一个toString\(\)方法覆盖的例子。

```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    //覆盖方法  
    @Override  
    public String toString() {  
        return this.index+"_"+this.name;  
    }  
}
```

**5、实现接口**

**所有的枚举都继承自**`java.lang.Enum`**类。由于Java 不支持多继承，所以枚举类不能再继承其他类**。

```text
public interface Behaviour {  
    void print();  
    String getInfo();  
}  
public enum Color implements Behaviour{  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    //接口方法  
    @Override  
    public String getInfo() {  
        return this.name;  
    }  
    //接口方法  
    @Override  
    public void print() {  
        System.out.println(this.index+":"+this.name);  
    }  
}
```

**6、使用接口的方式来组织枚举类**

```java
public interface Food { 
    enum Coffee implements Food{  
        BLACK_COFFEE,DECAF_COFFEE,LATTE,CAPPUCCINO  
    }  
    enum Dessert implements Food{  
        FRUIT, CAKE, GELATO  
    }  
}  

public static void main(String[] args) {
    Food coffee = Coffee.BLACK_COFFEE;
    Food dessert = Dessert.CAKE;
}
```

**7、关于枚举集合的使用**

`java.util.EnumSet`和`java.util.EnumMap`是两个枚举集合。EnumSet保证集合中的元素不重复；EnumMap中的key是enum类型，而value则可以是任意类型。

## 参考

[Java 枚举7常见种用法](http://blog.lichengwu.cn/java/2011/09/26/the-usage-of-enum-in-java/)

《Java编程思想》

