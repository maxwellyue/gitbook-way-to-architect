# 状态模式

状态模式用于解决系统中复杂对象的状态转换以及不同状态下行为的封装问题。

当系统中某个对象存在多个可以相互转换的状态，而且对象在不同状态下行为不相同时，可以使用状态模式。

状态模式将一个对象的状态从该对象中分离出来，封装到专门的状态类中，使得对象状态可以灵活变化，对于客户端而言，无须关心对象状态的转换以及对象所处的当前状态，无论对于何种状态的对象，客户端都可以一致处理。

状态模式\(State Pattern\)：允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。

状态模式的目的：将特定状态相关的逻辑分散到一些类的状态类中。

**状态模式中的角色**

* Context（环境类）：上下文类，其拥有多种状态的对象，维护一个抽象状态类State的实例，这个实例定义当前状态。
* State（抽象状态类）：定义一个接口以封装与Context的一个特定状态相关的行为，声明各种不同状态对应的方法，而在其子类中实现类这些方法，由于不同状态下对象的行为可能不同，因此在不同子类中方法的实现可能存在不同，相同的方法可以写在抽象状态类中。
* ConcreteState（具体状态类）：抽象状态类的子类，实现一个与Context的一个状态相关的行为，每一个具体状态类对应Context的一个具体状态，不同的具体状态类其行为有所不同。

> **小知识**：代码中state与status的区别
>
> **state所指的状态，一般都是有限的、可列举的，status则是不可确定的。**

**状态模式结构代码**

State

```java
public interface State{
    //声明抽象业务方法，不同的具体状态类实现不同    
    void handle(String parameter);
}
```

ConcreteState

```java
public ConcreteState implements State{

    public void handle(String parameter){
        System.out.print("具体的业务逻辑");
    }
}
```

Context

```java
public class Context{

    private State statee;

    public setState(Status state){
        this.state = state;
    }

    public void request(String arg){
        ...
        state.handle(parameter);
        ...
    }

}
```

举个例子：现在要开发一个屏幕放大镜工具：单击“放大镜”按钮之后屏幕将放大一倍，再点击一次“放大镜”按钮屏幕再放大一倍，第三次点击该按钮后屏幕将还原到默认大小。

Context：屏幕，客户端与Context交互

```java
public class Screen {  
    //枚举所有的状态，currentState表示当前状态  
    private State currentState, normalState, largerState, largestState;  

    public Screen() {  
        this.normalState = new NormalState(); //创建正常状态对象  
        this.largerState = new LargerState(); //创建二倍放大状态对象  
        this.largestState = new LargestState(); //创建四倍放大状态对象  
        this.currentState = normalState; //设置初始状态  
        this.currentState.display();  
    }  

    public void setState(Status state) {  
        this.currentState = state;  
    }  

    //单击事件处理方法，封转了对状态类中业务方法的调用和状态的转换  
    public void onClick() {  
        if (this.currentState == normalState) {  
            this.setState(largerState);  
            this.currentState.display();  
        }  
        else if (this.currentState == largerState) {  
            this.setState(largestState);  
            this.currentState.display();  
        }  
        else if (this.currentState == largestState) {  
            this.setState(normalState);  
            this.currentState.display();  
        }  
    }  
}
```

State

```java
public interface State{  
    void display();  
}
```

ConcreteState：有三个

```java
//正常状态
public class NormalState extends State{  
    public void display() {  
        System.out.println("正常大小！");  
    }  
}  

//二倍方法状态
public class LargerState extends State{  
    public void display() {  
        System.out.println("二倍大小！");  
    }  
}  

//四倍放大状态
public class LargestState extends State{  
    public void display() {  
        System.out.println("四倍大小！");  
    }  
}
```

客户端

```java
public class Client {  
    public static void main(String args[]) {  
        Screen screen = new Screen();  
        screen.onClick();  
        screen.onClick();  
        screen.onClick();  
    }  
}

//输出如下
正常大小！
二倍大小！
四倍大小！
正常大小！
```

整体逻辑是这样的：有一个对象（Origin Object），这个原始对象有不同的状态（其实就是Origin Object的某个属性的值为有限个，可以枚举），该对象中有一个方法doAction\(\)，这个方法根据Origin Object的状态的不同，实现逻辑不同，而状态模式则是将不同状态的特定行为逻辑转移到了不同的状态类中。

