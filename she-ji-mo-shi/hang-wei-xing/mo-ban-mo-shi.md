# 模板模式

## 概念与定义

在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。

即先定义一个抽象类，该抽象类中定义一个模板方法，在该模板方法中，定义算法的步骤，而每一个步骤中的具体实现，可以在该抽象类中实现，也可以交给子类去实现。

简单示例

```java
public abstract class Game {

    //模板方法
    public final void play(){

        //初始化游戏
        initialize();

        //开始游戏
        startPlay();

        //结束游戏
        endPlay();
    }

    //具体实现抽象出来交给子类实现
    abstract void initialize();
    abstract void startPlay();
    abstract void endPlay();
}
```

子类实现具体步骤：

```java
//足球
public class Football extends Game {

    @Override
    void initialize() {
        System.out.println("Football Game Initialized! Start playing.");
    }

    @Override
    void startPlay() {
        System.out.println("Football Game Started. Enjoy the game!");
    }

    @Override
    void endPlay() {
        System.out.println("Football Game Finished!");
    }
}

//羽毛球
public class Badminton extends Game {

    @Override
    void initialize() {
        System.out.println("Badminton Game Initialized! Start playing.");
    }

    @Override
    void startPlay() {
        System.out.println("Badminton Game Started. Enjoy the game!");
    }

    @Override
    void endPlay() {
        System.out.println("Badminton Game Finished!");
    }
}
```

## 模板的写法

* **模板方法**

  ：定义算法骨架的方法

* **具体的操作**

  ：在模板中直接实现某些步骤的方法。通常这些步骤的实现算法是固定的，而且是不怎么变化的，因此可以将其当做公共功能实现在模板中。如果不需要子类提供访问这些方法的话，还可以是`private`的。

* **具体的AbstractClass操作**

  ：在模板中实现某些公共功能，可以提供给子类使用，一般不是具体的算法步骤的实现，而是一些辅助的公共功能。

* **原语操作**

  ：就是在模板中定义的抽象方法，通常是模板方法需要调用的操作，是必须的操作，而且在父类中还没有办法确定下来，需要子类来真正实现的方法。

* **钩子操作**

  ：在模板中定义，并提供默认实现的操作。这些方法通常被视为可扩展的点，但不是必须的。

* **Factory Method**

  ：在模板方法中，如果需要得到某些对象实例的话，可以考虑通过工厂方法模式来获取，把具体构建对象的实现延迟到子类中去。

## 模板模式的本质

模板方法模式主要是通过制定模板，把算法步骤固定下来，至于谁来实现，模板可以自己提供实现，也可以由子类去实现，还可以通过回调机制让其他类来实现。  
通过固定算法骨架来约束子类的行为，并在特定的扩展点来让子类进行功能扩展，从而让程序既有很好地复用性，又有较好的扩展性。

模板方法模式很好地体现了设计原则中的开闭原则和里式替换原则。

内容文字摘抄自《研磨设计模式》

