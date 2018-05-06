# 外观模式

---

外观模式：通过引入一个外观角色来简化客户端与子系统之间的交互，为复杂的子系统调用提供一个统一的入口，降低子系统与客户端的耦合度，且客户端调用非常方便。

外观模式是一种使用频率非常高的结构型设计模式。

外观模式的本质：通过对象的封装，减小客户端使用复杂度。

外观模式中的角色：Facade和SubSystem。![](/assets/屏幕快照 2018-05-06 上午11.38.55.png)图（A）为未引入门面模式的情况；图（B）为引入Facade之后的情况。

**Facade模式结构代码**

`SubSystem`

```java
public class SubSystemA  
{  
    public void methodA()  
    {  
        //业务实现代码  
    }  
}  

public class SubSystemB  
{  
    public void methodB()  
    {  
        //业务实现代码  
     }  
}  

public class SubSystemC  
{  
    public void methodC()  
    {  
        //业务实现代码  
    }  
}
```

`Facade`：维持对子系统的引用

```java
public class Facade  
{  
    private SubSystemA obj1 = new SubSystemA();  
    private SubSystemB obj2 = new SubSystemB();  
    private SubSystemC obj3 = new SubSystemC();  

    public void method()  
    {  
        obj1.methodA();  
        obj2.methodB();  
        obj3.methodC();  
    }  
}
```

`Client`：通过外观类来间接调用子系统对象的方法，无需与子系统直接交互

```java
public class Client  
{  
    public static void main(string[] args)  
    {  
        Facade facade = new Facade();  
        facade.Method();  
    }  
}
```







